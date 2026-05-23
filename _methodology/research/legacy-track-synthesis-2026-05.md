# Legacy Track — research synthesis & decisions

> **Status:** consolidated decision register for the proposed Legacy audience-track. Synthesizes the four research files into Pavel's chosen framing (F2), seven implementation-level decisions (D1-D7), four open questions reserved for ratification (Q-*), and a draft OpenSpec proposal structure.
> **Owner:** Pavel (founder).
> **Opened:** 2026-05-13.
> **Companion to:** [`legacy-track-audience-2026-05.md`](./legacy-track-audience-2026-05.md), [`legacy-track-naming-2026-05.md`](./legacy-track-naming-2026-05.md), [`legacy-track-competitor-2026-05.md`](./legacy-track-competitor-2026-05.md), [`legacy-track-ia-funnel-2026-05.md`](./legacy-track-ia-funnel-2026-05.md).
> **Feeds into:** `openspec/archive/2026-05-22-add-legacy-track/` (proposal + design + tasks), Linear issue (TBD `RAP-NNN`).
> **Decision cutoff:** D1-D7 below are the pre-ratification baseline. Q-1..Q-4 remain open for Pavel's call before the change is committed.

---

## 1. Why this research happened

Pavel: *"Мне нужно выпустить legacy. Мое задание — сделать legacy, и для этого я хочу знать, что мне делать и как мне делать."*

There is no `/legacy` page, no Legacy service, and no Legacy reference in current specs except as a motivation axis in [`positioning.md § 5.1`](../positioning.md). Before any OpenSpec change is drafted, the framing and the wedge must be verified — otherwise the proposal would be improvisation.

Four research slices were dispatched in parallel on 2026-05-13: audience, naming, competitor landscape (legacy-positioning angle), and IA + funnel.

## 2. Framing decision — F2 (audience track), not F1 / F3 / F4

Pavel had four candidate framings on the table:

- **F1** — 6th service line, parallel to `discovery / migration / modular / platform / subscription`.
- **F2** — 3rd audience track, parallel to Client Track + Founder Track.
- **F3** — manifesto / editorial page.
- **F4** — sub-brand / new product line.

**F2 selected.** Reasoning:

1. **Structural symmetry.** Founder Track already runs the pattern "same 5 services, different lens, dedicated page, intake-source attribution". F2 reuses the pattern.
2. **Audience already mapped.** 4 of 9 archetypes in [`positioning.md`](../positioning.md) carry `legacy` as dominant motivation (A1 Second-Time Founder, A5 Consultant Seeking Leverage, A8 Solo Researcher, S1 Specialist-Publisher). No new persona work required.
3. **Doesn't force the service catalog.** "Legacy" is a mindset, not an engagement shape. F1 would have demanded a new shape with no operational backing.
4. **No new capability spec.** F4 would have required a months-long product-track effort (compare Canvas / Muse / Forge). F2 is page-scale.
5. **F3 is conversion-dead.** "Launch Legacy" implies a funnel; a manifesto-only deliverable has no intake.

## 3. Wedge confirmation — the most valuable single finding

From [`legacy-track-competitor-2026-05.md § 2`](./legacy-track-competitor-2026-05.md):

> "Second-time founder × paid fixed-engagement service" is an **uncontested copy wedge**. South Park Commons, Post-Exit Founders, The Flywheel, Hexa — all sit in adjacent slots (community, equity, studio-cofounder), none sell a fixed-engagement service to this cohort.

Adjacent unoccupied slots:

- **Knowledge stewardship / IP packaging (boutique tier)** — vocabulary cluster exists but only as productized-consulting self-help (Consulting Success, Philip Morgan). No verified studio sells "we package consultants' IP into durable digital products" white-glove.
- **Research-grade tools for solo experts** — closest analogue is Andy Matuschak (one person, Patreon-funded, now spinning out Pico). No service-studio incumbent in MD/RO/IL/EU.

In MD/RO/IL specifically, **fewer than 5 of ~80+ surveyed studios** touch legacy / long-arc vocabulary. The geographic field is structurally open.

## 4. Decisions (D1-D7) — what I commit to absent Pavel's override

Each decision is research-backed, ratifiable, and reversible.

### D1. Public name: keep "Legacy"

**Decision.** Keep "Legacy" as the EN public name, with an explicit anti-frame in hero copy ("Not the legacy code kind — the kind that's left behind"). In RO **avoid "Moștenire" as H1**; use English-loanword "Legacy" plus a non-death-coded Romanian subtitle. In RU, "Наследие" works positively.

**Why.** [`legacy-track-naming-2026-05.md`](./legacy-track-naming-2026-05.md) findings:

- ~7 live "Legacy [X]" agencies exist in EN, all mid-market branding. **No premium tech-tier occupant** — the wedge at the top of the EN field is available.
- Tech-debt overload is real (Stack Overflow 2024: 62% top-frustration; multiple industry pieces flag the term as pejorative for IT buyers). Reflexive anti-frame counters it.
- RO "Moștenire" is legally equivalent to "succesiune" — death-coded. Avoidance is mandatory.
- RU "Наследие" is solemn-positive (cultural / intellectual register); "Наследство" is death-coded — avoid.

**Risk acknowledged.** Tavern Agency and Untold use the "register split" pattern: "Heritage" externally, "Legacy" reserved for engineering. We are choosing to claim "Legacy" externally despite this — and Q-NAME-1 below leaves the door open to reverse.

### D2. IA: hybrid Founder-Track mirror + family-office content shift

**Decision.** Same 7-section count as `/founders` (internal consistency), but content shifted toward the family-office pattern. Working draft sequence:

1. Hero (legacy-frame with anti-tech-debt subtitle)
2. Philosophy — what we believe about building for the long arc *(new; replaces founder-stories section)*
3. Four archetypes — A1 / A5 / A8 / S1 as use-case cards, each with the existing 5 services lensed for that archetype
4. How we work — same 4-step Discovery as `/founders`, but emphasizing craft over speed
5. Trust pillars — Authorship / Security / Portability (same canonical sentences as `/founders`)
6. People — Pavel + named network *(new; family-office IA pattern)*
7. Soft CTA — "Schedule an introduction" (not "Send brief")

**Why.** [`legacy-track-ia-funnel-2026-05.md § 2`](./legacy-track-ia-funnel-2026-05.md): family-office IA (Rockefeller, Crosby, Gresham, Matter, White Oak) is the verified pattern for long-arc / UHNW audiences; explicitly NOT lead-gen-first. Pure Founder Track mirroring would be lead-gen-heavy and miss the audience register. Pure family-office adoption would create an unjustified IA asymmetry vs `/founders`. The hybrid keeps section count + skeleton, swaps content emphasis.

### D3. Lead archetype: A1 (Second-Time Founder)

**Decision.** Page hero copy speaks to **A1 primarily**. A5 (consultant-author), A8 (solo researcher), S1 (specialist-publisher) appear as secondary use cases in the archetype-cards section but do not lead.

**Why.**

- [`legacy-track-competitor-2026-05.md § 2`](./legacy-track-competitor-2026-05.md): A1 × paid engagement is the strongest verified empty market.
- [`legacy-track-audience-2026-05.md § 1`](./legacy-track-audience-2026-05.md): IL cyber-second-time-founder economy is real (Calcalist); RO Forbes-tech-money cohort is real (Dines, Stoica, Zaharia); both fall under A1.
- €50k floor + 6-month minimum is compatible with the A1 budget envelope; less reliable for A8 (solo researcher) without grant backing.
- Trying to address 4 archetypes equally would dilute hero copy to abstract platitudes.

### D4. Geographic asymmetry: locale-specific copy is mandatory

**Decision.** EN canonical, RU and RO each get **authored**, not translated, copy. Each locale runs a semantically distinct register:

- **EN canonical** — "long arc" / "built to stay" / "second chapter" voice. Closest reference: 37signals + Edenspiekermann.
- **RO** — family-business-continuity reality (Civil Code forces inheritance share, so succession is culturally baseline), but **soft**. No "for your children" framing — the cultural association is too forced. Lean into "what outlasts the engagement".
- **RU** — "Наследие" works positively; pair with craft + patient + stewardship vocabulary.
- **(deferred) IL-EN** — second-chapter / post-cyber-exit framing. **NOT generational.** Captured in Q-IL-1 below.

**Why.** [`legacy-track-audience-2026-05.md § 4`](./legacy-track-audience-2026-05.md): the geographic registers are not lexically different, they are semantically different. RO Civil Code makes succession unavoidable; IL post-cyber-exit cohort reads "for your children" as off-key. A single translated copy would fail at least one geo.

### D5. Intake: shared `/contact` + `?source=web/legacy`

**Decision.** No separate intake form. Re-use the existing `IntakeForm` at `/contact` with `?source=web/legacy` appended to the source allowlist at [web.md:205](../../current/web.md).

**Why.** [`legacy-track-ia-funnel-2026-05.md § 3`](./legacy-track-ia-funnel-2026-05.md): split-intake lift (~68%) is documented only with matched ad traffic, not for organic / audience-page traffic at low absolute volume. Founder Track already runs on shared-intake-with-attribution; breaking the pattern adds maintenance cost without commensurate conversion proof.

### D6. Nav placement: NOT in main nav

**Decision.** Discovery via:

- Homepage block `<ForLegacy />` (sibling to existing `<ForFounders />`)
- Footer link
- Cross-link from `/lab` articles (where editorial content speaks to A1 / A5 / A8 / S1)
- JSON-LD `Service` entries reference `/legacy` for SEO indexing

**Why.** [`legacy-track-ia-funnel-2026-05.md § 5`](./legacy-track-ia-funnel-2026-05.md): NN/G hard rule — audience-based nav splits fail unless content is genuinely unique to the audience; when justified, placement is utility nav, not main nav. The existing `/founders` page also is not in main nav (per [`home/blocks.md § Nav`](../../current/home/blocks.md)). Mirror precedent.

### D7. i18n at day 1: en + ru + ro, all authored

**Decision.** Ship `/legacy` in three locales on day 1, each with authored copy per D4. No machine translation. i18n keys live under `legacy.*` namespace.

**Why.** [`legacy-track-ia-funnel-2026-05.md § 6`](./legacy-track-ia-funnel-2026-05.md): retrofit i18n is "messy and pricey" (Lokalise, SimpleLocalize); industry norm is single-locale launch but with i18n routing in place from day 1. Pavel's site already runs en/ru/ro for Founder Track; deviating creates inconsistency. Per D4, machine translation breaks the geographic register strategy — authored copy is mandatory.

## 5. Open questions reserved for Pavel (Q-1..Q-4)

These are real trade-offs I refuse to decide. They land in `design.md` § Open questions and gate ratification.

### Q-NAME-1 — Rename away from "Legacy"?

Trade-off: keep "Legacy" (positioning-link preserved, premium-tech wedge claimed) vs rename (escape tech-debt overload, but vocabulary is fragmented across alternatives).

Unclaimed alternatives at premium tier (per [`legacy-track-naming-2026-05.md § 5`](./legacy-track-naming-2026-05.md)): **Steward, Perennial, Long arc, Long view, Continuity, Enduring**. No live agency owns these — either a positioning opportunity or a signal of weak naming juice. Pavel decides.

### Q-PRICE-1 — Publish €50k floor / 6-month minimum on `/legacy`?

Per [`legacy-track-competitor-2026-05.md § 5`](./legacy-track-competitor-2026-05.md), **0 of ~10 surveyed legacy-positioned studios publish pricing**. Hiding pricing is segment norm. Pavel's existing `/services` page does publish the floor. Should `/legacy` mirror `/services` (consistency) or align with the segment (legacy-norm)?

### Q-MANIFEST-1 — Add `/legacy/manifesto` as a separate long-form document?

37signals (their 37 signed principles), Long Now ("framework of the next 10,000 years"), Mule ("Last of the independents"), INDIO (founding manifesto) — all legacy-positioned firms maintain a separate manifest document beyond the marketing page. It is a segment pattern.

Cost: one additional MDX file in `apps/web/src/content/`, one route, ongoing maintenance. Pavel-only capacity at present. Worth it now or deferred to post-launch?

### Q-IL-1 — Separate IL-EN copy lens or shared EN?

Per D4 + [`legacy-track-audience-2026-05.md § 4`](./legacy-track-audience-2026-05.md), the IL post-cyber-exit register is semantically different from generational-EN. Founder Track currently ships single Israel-first EN copy without a separate locale — should Legacy mirror that (one shared EN) or carry a separate `?lens=il` query-driven variant?

## 6. OpenSpec proposal — proposed structure

Change slug: **`add-legacy-track`** (bare slug while active per [`feedback_openspec_folder_naming`](file:///Users/pavelrapoport/.claude/projects/-Users-pavelrapoport-Documents-GitHub-rapoport-studio/memory/feedback_openspec_folder_naming.md); date prefix only on archive).

Triplet:

```
openspec/archive/2026-05-22-add-legacy-track/
├── proposal.md
├── design.md
└── tasks.md
```

### proposal.md outline

- Why now (market wedge + cultural moment 2025-26)
- What we ship (3-locale `/legacy` route + homepage block + footer link + intake source allowlist update)
- Who it's for (A1 primary; A5 / A8 / S1 secondary)
- Scope / non-scope (NOT a new service, NOT a sub-brand, NOT a redesign of `/services`)
- Success criteria (intake-source attribution data + post-launch funnel observability)

### design.md outline

- §1 — Decision: F2 framing with research provenance (links to these 4 research files)
- §2 — Naming decision (D1) + Q-NAME-1
- §3 — IA: 7-section hybrid (D2)
- §4 — Section-by-section copy framework (eyebrow / heading / lede / body constraints, modeled on [web.md `/founders`](../../current/web.md))
- §5 — Lead archetype A1 (D3); secondary archetype handling
- §6 — Geographic strategy: locale-specific authored copy (D4)
- §7 — Forbidden vocabulary list (modeled on [web.md:219](../../current/web.md)): "AI-powered" / "10x" / "move fast" / "magic button" / "MVP" + per-locale additions
- §8 — Positive vocabulary palette: craft / patient / stewardship / built to stay / second chapter / long arc + per-locale variants
- §9 — Intake (D5), nav (D6), i18n (D7)
- §10 — Open questions: Q-NAME-1, Q-PRICE-1, Q-MANIFEST-1, Q-IL-1
- §11 — Research provenance: links to 4 sibling research files

### tasks.md outline — 7 phases

1. Spec ratification (close Q-1..Q-4)
2. Copy authoring × 3 locales (en canonical first; ru + ro authored per D4)
3. Route scaffold (`/[locale]/(landing)/legacy/page.tsx` + `generateMetadata` + `generateStaticParams`)
4. Components (hero, philosophy, archetype cards, process, trust pillars, people, soft CTA)
5. Homepage `<ForLegacy />` block (sibling to `<ForFounders />`) + footer link
6. Intake source allowlist update (`web/legacy` → [web.md:205](../../current/web.md))
7. SEO + JSON-LD `Service` entries with legacy lens + sitemap + hreflang; QA; ship

## 7. Provenance, dating, decay

- **Research conducted:** 2026-05-13.
- **Shelf life:** competitive landscape data 6-12 months (per `competitor-landscape-synthesis.md § 1`); audience research 12-18 months; naming research evergreen modulo cultural shifts; IA / funnel best-practices 12-24 months.
- **Re-trigger:** if launch slips past 2026-12-31, refresh `legacy-track-competitor` slice. If a major AI-backlash inflection passes (current Q4-2025 trough), refresh `legacy-track-audience § 3` vocabulary-that-repels.
- **Not covered:** validated conversion data for `/founders`-style audience tracks vs generic `/services`; %-of-MD/RO founders who are second-time; verified buyer-hostility to "growth hacking" as a phrase.
