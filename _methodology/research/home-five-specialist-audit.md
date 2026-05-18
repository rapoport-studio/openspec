# Home — Five-Specialist Audit, Repo-Verified Findings

> **Status:** standalone structural research. NOT tied to a ratified change.
> **Branch:** `docs/home-decorative-zone-audit` — research only, **zero code edits**.
> **Audited:** the marketing home page + `/work`, EN/RU, of `rapoport.studio`.
> **Date:** 2026-05-18.
> **Sibling:** `home-decorative-zone-audit.md` (decorative-zone structural pass).

---

## 1. Method note — and why the source audit could not be trusted

A five-specialist live-site audit (frontend / brand / conversion / SEO / IA) was
run against `rapoport.studio` and produced ~22 findings. That audit was generated
by **`web_fetch` of the live deployment** — which is not ground truth:

- `web_fetch` returns whatever the edge serves at fetch time, which may be a
  **stale or preview deploy**, not current `main`.
- `web_fetch` **strips `<script>` tags**, so every JSON-LD / structured-data
  claim in the source audit was a guess.

This document re-checks every concrete claim **against repo code at a cited
`file:line`**. Result: of ~22 source findings, **6 survive verification**; the
rest are refuted (§3) or unverifiable without a browser/DB (§4).

### Why was the live fetch stale? (open risk)

The source audit reported bugs that **do not exist in `main`** — localhost OG
URLs, an untranslated RU meta description, missing RU OpenGraph tags, a
`/contact` route. All are correct in the repo today (§3). The most probable
cause is that the fetch hit a **stale artifact** — candidates, none confirmed:

- **Cloudflare edge cache** serving a pre-fix HTML snapshot;
- a **preview / non-production deploy URL** (OpenNext preview worker);
- **OpenNext SSR/ISR caching** holding an old render (`revalidate = 3600` is set
  on the home route — `(marketing)/page.tsx:1`).

**This is an unresolved risk.** Until the staleness cause is identified, the same
failure mode will recur in the next live audit. The durable fix is a
verification-gate methodology rule — successor change 4 in §5.

---

## 2. Confirmed findings

Format per finding: **Conflict** → **Recommendation** → **Related finding to
verify** → *no-code-change note*.

### F1 — Cast naming surfaces in public marketing copy

**Conflict.** `openspec/_methodology/manifesto.en.md:40` states the Cast names
are *"team-internal only — they never appear in public branding, client
deliverables, or marketing copy."* The Cast table (`manifesto.en.md:42-49`)
lists six: **Maestro, Muse, Atlas, Echo, Pulse, Forge**.

The home **Operating Model** section renders all six to users. The node labels
live in `packages/i18n/messages/{en,ru,ro}/web.json` under
`home.operatingModel.nodes.*.label`, consumed by
`apps/web/src/components/home/operating-model.tsx:21-29` and rendered by
`operating-model-diagram.tsx:42-48`. Verified EN values:

| node key | `label` | `oneliner` |
|---|---|---|
| `maestro` | `Maestro` | `coordinator · auditor` |
| `muse` | `Muse` | `architect · drafter` |
| `forge` | `Forge` | `builder · PR machine` |
| `atlas` | `Atlas` | `research agent` |
| `echo` | `Echo` | `voice interface` |
| `pulse` | `Pulse` | `financial agent` |
| `mirror` | `Mirror` | `knowledge layer` |
| `linear` | `Linear` | `issues · planning` |
| `github` | `GitHub` | `your repo · your commits` |

The same Cast names also appear in the **ConnectedStack** tile descriptions
(see F2): `connected-stack.tsx:48,57,66,84,92` reference "Forge" and "Muse" in
user-facing copy. F1 is therefore not limited to the diagram.

**Decision record — D-FSA-1.** Cast names → role descriptors. **Ratified by
Pavel, 2026-05-18.** All six Cast names are removed from public marketing copy;
`manifesto.en.md` is left unchanged (the policy stands). This block is the
source of truth for the decision — successor changes cite `D-FSA-1`.

**Recommendation.** Replace the six Cast-name `label` values with role
descriptors. The existing `oneliner` strings already *are* role descriptors
(`coordinator · auditor`, `architect · drafter`, `builder · PR machine`,
`research agent`, `voice interface`, `financial agent`) — the successor change
can promote a short form of each `oneliner` into the `label` slot, in all three
locales, and re-flow the `oneliner`. Scope notes:

- `linear` / `github` are real third-party product names — **keep**.
- `mirror` is **not** one of the six Cast names (`manifesto.en.md:42-49`); it is
  a package/component name (`@rapoport-studio/muse/mirror`, per CLAUDE.md
  boundary table) and reads as a role ("knowledge layer") — **out of F1 scope,
  keep as-is**.
- The ConnectedStack descriptions (F2) must also drop "Forge"/"Muse" when they
  are i18n-extracted — fold the D-FSA-1 rewrite into that work.

**Related finding to verify.** Sweep other marketing surfaces (`/founders`,
`/services`, `/network`, `/discovery`) for Cast names before the successor
change closes — this audit only confirmed the home page.

*No code change in this branch. Page-level fix lands in the Cast-naming
successor change (§5 item 2).*

### F2 — ConnectedStack tile descriptions are hardcoded English

**Conflict.** `apps/web/src/components/home/connected-stack.tsx:41-120` — the
`CONNECTIONS` array's nine `description` fields are **English string literals**,
not `t()` i18n lookups. `ConnectionTile` renders `{connection.description}` raw
(`connected-stack.tsx:174-176`). The `label`, `functionKey`→`functions.*`, and
status labels *do* go through `t()`; only `description` does not. Result: the
nine descriptions render English on **every** locale (`/ru`, `/ro` included).
The nine literals:

```
github     "Code, PRs, and commits — Forge works in your repository."
linear     "Tasks and cycles — Forge plans and builds against your backlog."
telegram   "Per-canvas Muse surface — voice notes, decisions, approvals."
cloudflare "Deploys, Workers, R2, and DNS — infrastructure visible in one view."
figma      "Muse reads design files and proposes code from components."
anthropic  "Claude API — the substrate Muse runs on."
stripe     "Billing and invoicing for studio engagements."
supabase   "Postgres, Auth, and Storage — the studio data plane."
infisical  "Secrets management for every connection above."
```

**Recommendation.** Extract to
`home.connectedStack.connections.<slug>.description` keys in
`packages/i18n/messages/{en,ru,ro}/web.json`; update `connected-stack.tsx` to
read `t(...)`. EN values = the current literals **minus Cast names** (per
D-FSA-1 — `github`/`linear`/`telegram`/`figma`/`anthropic` copy mention "Forge"
/ "Muse"); RU/RO need real translations. The cross-locale parity test
(`packages/i18n/src/__tests__/smoke.test.ts`, per `web.md:138`) will then cover
these keys.

**Related finding to verify.** Audit other home components for the same
hardcoded-literal pattern (any user-facing string not behind `t()`).

*No code change in this branch. Fix lands in the marketing-i18n successor
change (§5 item 3).*

### F3 — Home "Now building" cards link to a deferred route → 404

**Conflict.** `apps/web/src/components/home/now-building.tsx:40` links each card
via `href={ROUTES.workCase(card.slug)}` → `/work/<slug>`. `/work/[slug]` is
**v1.x-deferred**: `openspec/current/web.md:104` (routing table:
`/[locale]/work/[slug]` … **v1.x-deferred**), `:90`, `:171` ("`/work/[slug]`
detail page remains v1.x-deferred"). `packages/i18n/src/routes.ts:65-67` lists
`workCase` under `PENDING_ROUTES` with the comment *"known broken link until …
the page is built or the calling site is changed."* So the first interactive
card after the hero leads to a 404.

The source audit framed this as consistent with an accepted "404 chrome handles
it" decision. **That premise is wrong.** `web.md:123` says deep-link routes
*activate* when the corpus crosses a size heuristic (3+ entries) — there is **no
documented decision to ship clickable links to a deferred route in the
meantime**. The code comment in `work/page.tsx:24-26` asserting that 404s are an
accepted interim state is itself **drift**, not a precedent.

**Recommendation.** Render the "Now building" cards as **non-clickable preview
tiles** (no `href`) until `/work/[slug]` ships; activate the links in the same
change that ships the detail route.

**Related finding to verify.** See F4 — the `/work` index has the same dead
link; it is a second instance, not a justification.

*No code change in this branch. Fix lands in `fix-deferred-route-deadlinks`
(§5 item 1).*

### F4 — `/work` index cards have the same dead link (separate instance)

**Conflict.** `apps/web/src/app/[locale]/(marketing)/work/page.tsx:103` (`<FeaturedWorkTile
… href={ROUTES.workCase(entry.slug)} />`) and `:131` (`<WorkCard …
href={ROUTES.workCase(entry.slug)} />`) — both zones of the `/work` index link
to the same v1.x-deferred `/work/[slug]` route. This is a **second occurrence**
of F3, not its precedent.

**Recommendation.** Same as F3 — non-clickable until the detail route ships.
Fold F3 + F4 into one successor change so home and `/work` activate together.

*No code change in this branch.*

### F5 — JSON-LD missing on `/discovery` and `/specialists`

**Conflict.** `apps/web/src/app/[locale]/(marketing)/discovery/page.tsx` and
`specialists/page.tsx` both define `generateMetadata` but emit **no**
`<script type="application/ld+json">` (verified: `grep -c "ld+json"` → `0` for
both). The home, `/work`, `/lab`, `/legacy`, `/founders`, `/services` pages all
emit JSON-LD. This is **not cosmetic**: `/discovery` is the entry-tier offer and
`/specialists` is the community track — both are conversion-critical surfaces,
and missing structured data directly undercuts the structured-data /
AI-citation thesis behind `expand-marketing-surface-stage-1`.

**Recommendation.** Add JSON-LD via the existing helpers in
`apps/web/src/lib/jsonld.ts` — `/discovery` maps to a `Service` entry
(`serviceLdEntries` already has a `discovery` record, `jsonld.ts:5-9`);
`/specialists` fits `collectionPageLd` (`jsonld.ts:85-98`).

*No code change in this branch. Fix lands in the marketing-i18n successor
change (§5 item 3).*

### F6 — `operating-model-proof.tsx` localhost fallback (latent, low)

**Conflict.** `apps/web/src/components/home/operating-model-proof.tsx:6-10` —
`resolveBaseUrl()` falls back to `http://localhost:3000` when neither
`NEXT_PUBLIC_SITE_URL` nor `VERCEL_URL` is set. It is used only for an internal
stats `fetch` (`/api/operating-model/stats`, `:13`), **never** for OG metadata —
so the source audit's "localhost in OG image" P0 is refuted (§3). The fallback
is still a latent bug: on a runtime where neither env var is set, the stats
fetch would target localhost and silently fail (the component then returns
`null`, `:30`).

**Recommendation.** Replace the localhost fallback with the canonical
`https://rapoport.studio` (or resolve the Worker request origin). Low priority —
bundle with §5 item 3 if convenient.

*No code change in this branch.*

### Note — `web.md` self-inconsistency (spec drift, not a page bug)

`openspec/current/web.md:153` (§ Page contracts, homepage item 4) says "Now
building … pulls from `content/work/*.mdx`", but work content is now
**Supabase-backed** (`web.md:171,256` — `work_engagements` table via
`apps/web/src/content/work/loader.ts`; no `.mdx` files exist in
`apps/web/src/content/work/`). Minor `web.md` internal drift — flag for whoever
next amends `web.md`.

---

## 3. Refuted claims

Source-audit claims disproved by repo code. Each row cites the **specific
`file:line` that disproves it**.

| Source claim | Verdict | Disproved by |
|---|---|---|
| localhost in `og:image` / `twitter:image` | **Refuted** | `(marketing)/page.tsx:35` hardcodes `baseUrl = "https://rapoport.studio"`; OG URLs built from it at `:77`. The only `localhost` is `operating-model-proof.tsx:9` — an internal API fetch, not metadata (see F6). |
| RU `meta description` is English | **Refuted** | `packages/i18n/messages/ru/web.json` → `home.meta.description` is a real RU translation ("AI-native платформы за недели…"). EN/RO likewise translated. |
| RU page missing `og:image` / `canonical` / `og:locale:alternate` | **Refuted** | `(marketing)/page.tsx` `generateMetadata` (`:52-95`) sets `openGraph.images` (`:77`), `alternates.canonical` (`:86`), `alternates.languages` (`:87-92`), `openGraph.locale`+`alternateLocale` (`:62-76`) — per-locale, all locales. |
| `/start-project` ↔ `/contact` route drift between EN and RU | **Refuted** | `packages/i18n/src/routes.ts:32` — single `startProject: "/start-project"`, no per-locale variants; `packages/i18n/src/routing.ts` has no `pathnames` map; `hero.tsx:76` uses `ROUTES.startProject` for all tracks. No `/contact` route directory exists. (`web.md:218` notes `/contact` was *renamed* to `/start-project`.) |
| RU footer missing DPA + Sub-processors | **Refuted** | `packages/i18n/messages/ru/common.json` → `footer.legal.dpa` and `footer.legal.subProcessors` both present; EN/RO parity holds. |
| Founder-shape markets diverge EN vs RU | **Refuted** | `packages/i18n/messages/{en,ru}/web.json` — `home.forFounders.segments.*.geo` and `home.whoWeWorkWith.cohorts.*.geo` are byte-identical across EN and RU. |
| RU hero has an extra Cast-name sentence | **Refuted** | `packages/i18n/messages/ru/web.json` → `home.hero.sub` / `home.hero.subWithFounders` are parallel to EN; neither names Muse/Forge/Maestro. |
| Connections + Operating Model render twice = bug | **Refuted (non-issue)** | `connected-stack.tsx:210` (`md:hidden` mobile block) + `:239` (`hidden md:grid` desktop block) — a deliberate responsive split. `display:none` removes the inactive variant from the accessibility tree; standard pattern, no `aria-hidden` needed. |
| `llms.txt` / `llms-full.txt` missing | **Refuted** | Both ship as static files in `apps/web/public/` (`llms.txt`, `llms-full.txt`); generated by `tools/build-llms-full.mjs`. |

Probable root cause for the whole table: the live fetch hit a stale/preview
deploy (see §1 — open risk).

---

## 4. Unverifiable items

Out of scope for a code audit — need a browser, the Supabase DB, or runtime
knowledge:

- **Stealth work slug leaking client identity.** Work is Supabase-backed
  (`work_engagements` table); no `.mdx` work files exist in the repo. Slugs live
  in the DB — cannot inspect without DB access.
- **Telegram bot liveness.** The ConnectedStack tile marks Telegram
  `status: "connected"` and `closing-cta.tsx` links `studio.telegramUrl` — but
  whether the bot actually responds is a runtime fact.
- **Lighthouse / Core Web Vitals**, **accessibility tree / focus order /
  contrast**, **mobile render**, **`prefers-reduced-motion` on `TacticalFrame`**
  — all need a real browser run.

---

## 5. Recommended successor changes

Named, **not created** here. Dependency order:

1. **`fix-deferred-route-deadlinks`** — F3 + F4. Independent, cheapest, visible
   user-facing fix; render home + `/work` cards non-clickable until
   `/work/[slug]` ships, and re-activate the links in the change that ships the
   detail route. Start first.
2. **Cast-naming change** — F1 (Operating Model labels + ConnectedStack
   descriptions → role descriptors). Decision **D-FSA-1** already made; needs
   only the i18n update. Blocks any future branding pass on the Operating Model
   section, so do it before such a pass.
3. **`marketing-i18n-pass`** — F2 (+ F5, F6). Independent, non-blocking, widest
   surface; can run in parallel with 1–2. Note F2's EN values depend on D-FSA-1
   (descriptions mention Cast names), so coordinate with change 2.
4. **`add-audit-discipline`** (separate, methodology). Codify a gating rule in
   `openspec/_methodology/audit-discipline.md`: any live-fetch audit asserting
   structural/semantic findings (i18n parity, metadata, routes) requires a
   repo-verification pass before findings reach Linear or a PR. This is the
   durable fix for the §1 staleness risk — out of scope for this research doc,
   recorded here as the methodological follow-up.

---

## 6. Provenance

Every claim traces to a file inspected during verification:

- `openspec/_methodology/manifesto.en.md`, `openspec/current/web.md`
- `apps/web/src/app/[locale]/(marketing)/page.tsx`, `work/page.tsx`,
  `discovery/page.tsx`, `specialists/page.tsx`
- `apps/web/src/components/home/`: `operating-model.tsx`,
  `operating-model-diagram.tsx`, `operating-model-proof.tsx`,
  `connected-stack.tsx`, `now-building.tsx`, `hero.tsx`
- `apps/web/src/lib/jsonld.ts`
- `packages/i18n/src/routes.ts`, `routing.ts`;
  `packages/i18n/messages/{en,ru,ro}/{web,common}.json`
- `apps/web/public/llms.txt`, `llms-full.txt`; `tools/build-llms-full.mjs`

Verification was performed by reading the above; no live site was trusted.
