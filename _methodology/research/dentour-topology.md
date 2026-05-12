# Dentour Topology — Pre-engagement Scout

<!-- openspec-refs: allow-unresolved -->
<!-- This document describes Dentour's repository internals. All `openspec/changes/<slug>/`, `openspec/specs/`, etc. references describe paths inside `dentour-org/dentour`, NOT this repo. They are not actionable refs from the studio side, so the openspec-refs guard skips this file. Mirrors the same opt-out applied to vivod-topology.md (PR #767). -->

> Studio-internal topology scan of `dentour-org/dentour`, anchored to a known commit. Captured during Phase 1 of [`onboard-dentour`](../../changes/onboard-dentour/). Read-only — nothing in this document was committed back to Dentour's repo.

```yaml
scout_date: 2026-05-12
dentour_commit_sha: 2e96f677b7557b59530fbae87387b28e088defe3
dentour_commit_subject: 'feat(workspace): bootstrap openspec workflow with wave a baselines (2026-04-30)'
dentour_repo_url: https://github.com/dentour-org/dentour
governance_files_read:
  - CLAUDE.md
  - .github/copilot-instructions.md
  - BRANCHING_STRATEGY.md
  - README.md (badges confirm dentour-org/dentour)
  - openspec/INDEX.md
  - openspec/config.yaml
  - openspec/specs/*/spec.md (12 capabilities — 7 stable, 5 Phase-0 skeleton)
  - package.json (root + per-app/per-package selections)
  - pnpm-workspace.yaml
  - turbo.json
  - commitlint.config.js
  - .github/workflows/* (17 files inventoried)
shareability: studio-internal-default
author: forge-subagent
companion_docs:
  - dentour-drift-audit.md (Phase 2 — not yet authored)
  - dentour-convention-adapter-design.md (Phase 6 — not yet authored)
```

---

## 1. Repo identity and source of truth

- **Canonical repo:** `dentour-org/dentour` (confirmed via [`README.md`](https://github.com/dentour-org/dentour/blob/main/README.md) CI badges: `https://github.com/dentour-org/dentour/actions/workflows/ci.yml/badge.svg` and `https://img.shields.io/github/v/release/dentour-org/dentour`).
- **Production URLs:** seven country domains served from one codebase — `dentour.eu` (EU/EUR, default), `dentour.co.il` (IL/ILS, Hebrew RTL), `dentour.md` (MD/MDL), `dentour.ro` (RO/RON), `dentour.ge` (GE/GEL), `dentour.es` (ES/EUR), `dentour.co.uk` (UK/GBP). Operational subdomains: `clinic.dentour.eu` (clinic portal), `admin.dentour.eu` (admin), `api.dentour.eu` (API), `staging.<domain>` + `api-staging.dentour.eu` + `clinic-staging.dentour.eu` for the staging environment.
- **Branching model:** `feat/* | fix/* | chore/* | docs/* → develop → main`. Develop is the integration branch; merging develop → main triggers the production deploy + Changesets release flow. Hotfixes branch from `main` and back-merge to `develop`. Source: [`BRANCHING_STRATEGY.md`](https://github.com/dentour-org/dentour/blob/main/BRANCHING_STRATEGY.md).
- **Commit convention:** `type(scope): subject` enforced by `commitlint.config.js` (extends `@commitlint/config-conventional`). Type enum: `feat | fix | docs | style | refactor | perf | test | build | ci | chore | revert`. Scope enum is severity 2 (errors on miss) with 30 allowed values across Apps / Packages / cross-cutting features / Infrastructure. Subject must be lowercase (severity 2), no trailing period (severity 2), header ≤100 chars (severity 2). **There is no studio-side scope** — commits scoped `(onboard-dentour)` will hard-fail commitlint. Either register a new scope or land via a permitted scope (`docs`, `workspace`, or one of the cross-cutting feature scopes).
- **Package manager:** `pnpm@9.15.0` (pinned via `packageManager` field). Node engine `>=20.0.0`.
- **Monorepo orchestrator:** Turborepo (devDep `turbo ^1.11.0` — note: still v1.x, **not v2 like Vivod**). Remote cache enabled (`remoteCache.enabled: true` in `turbo.json`; `TURBO_TOKEN` + `TURBO_TEAM` referenced by CI). Pipeline (legacy v1 syntax: `pipeline.*`, not `tasks.*`) defines `build / dev / lint / type-check / clean / e2e` plus a handful of per-package overrides (notably `@dentour/api#build` depends on `@dentour/database#db:generate`).
- **OpenSpec tooling:** `openspec` CLI invoked via root scripts `pnpm openspec:list` and `pnpm openspec:validate --all --strict`. **CLI is not declared as a devDependency in root `package.json`** — assumed globally installed or invoked via `npx` (the script just runs `openspec` directly). Validation is **not wired into CI** (see § 6); it is a local-only step invoked by the `/opsx:apply` and `/opsx:archive` Claude-skill commands.
- **Versioning:** Changesets (`@changesets/cli ^2.29.7`, `@changesets/changelog-github ^0.5.1`). `pnpm changeset` + `pnpm changeset:version` + `pnpm changeset:publish` round-trip; release workflow runs on merge to `main`.

---

## 2. Deployed apps inventory

Dentour ships **three deployment classes**: three Cloudflare-Pages-hosted Next.js frontends (one with a parallel Cloudflare-Workers SSR path active in staging only), one Railway-hosted NestJS API, plus two non-deployed support apps (`e2e-tests`, `mcp-server`), one CLI-only app (`ai-team`), and one Fumadocs documentation site. There is one vestigial folder (`auth-portal`) with no `package.json`.

### 2.1 Patient-facing Next.js — `apps/patient-web` (`patient-web`, package version `0.11.0`)

- **Framework:** Next.js 15.5 with React 19, TypeScript 5.3+, Tailwind CSS 3.4. Auth via `@supabase/ssr ^0.8` + `@supabase/supabase-js ^2.86`. Data fetching: TanStack Query 5 (`@tanstack/react-query ^5.90`).
- **Routing:** App Router under `src/app/[locale]/`. **Locale segment values present in `messages/`:** `en`, `es`, `he`, `ka`, `ro`, `ru` (six locales). The governance docs (CLAUDE.md § 1, `openspec/config.yaml`, `.github/copilot-instructions.md`) all describe four locales (`en | ru | ro | he`); Spanish and Georgian were added to support `dentour.es` and `dentour.ge` but the governance docs were not updated. **Flag for kickoff.**
- **Dual build target — *both* paths exist in CI:**
  - **Static export → Cloudflare Pages (production active path).** `pnpm pages:build` runs `STATIC_EXPORT=true next build`; `wrangler pages deploy out` to a Cloudflare Pages project per region. Used by `deploy-production.yml` (main → prod) and `deploy-staging.yml` (develop → staging). Sourcemaps uploaded to Sentry. No SSR at runtime.
  - **OpenNext SSR → Cloudflare Workers (staging-only path).** `pnpm opennext:deploy:staging` runs `opennextjs-cloudflare build && wrangler deploy --config wrangler.workers.toml --env staging`. Wired in `deploy-staging-ssr.yml`. **Not used for production.** Same seven-region matrix.
  - **Drift:** CLAUDE.md § 1, `openspec/config.yaml` § context, and `.github/copilot-instructions.md` § Overview all assert "patient-web deploys to Cloudflare Workers (SSR via OpenNext)". Reality at HEAD: production is **static export on Pages**. The SSR path is a staging-only experiment. This is the largest single drift to flag in the topology.
- **Domain matrix:** every region gets its own Cloudflare Pages project (`dentour-production-{eu,il,md,ro,ge,es,uk}`) with locale env-var pinned per region (`en, he, ro, ro, ka, es, en`). 7-region matrix is fanned out in `strategy.matrix.include` and deploys are gated `max-parallel: 1` in production.
- **Observability:** Sentry (`@sentry/nextjs ^10`), client instrumentation file `apps/patient-web/instrumentation-client.ts`. Sourcemaps uploaded by CI to `dentour` org, `patient-web` project. Per CLAUDE.md § 8 the sourcemap upload is the "fixed" version of an earlier disabled-by-default state.
- **i18n:** `next-intl ^4.0` plus `@dentour/i18n` for shared domain data (services, specialties). Hebrew (`he`) is RTL.

### 2.2 Clinic-facing Next.js — `apps/clinic-portal`

- **Framework:** Next.js 15 (App Router), React 19, port 3003 in dev.
- **Deploy:** **Cloudflare Pages (static export).** Workflow `deploy-clinic-portal.yml` triggers on push to main/develop (and PRs) for `apps/clinic-portal/**`, `packages/{ui,types,utils,maps,observability,tailwind-config}/**`. Uses `wrangler pages deploy`. CLAUDE.md / copilot-instructions describe this as "static export in production" — confirmed.
- **Auth:** `@dentour/auth` (Supabase singleton), `AuthProvider` + `RoleGuard` model, `RoleGuard allowedRoles={['CLINIC_OWNER', 'CLINIC_STAFF']}`.
- **Intercom:** wired client-side via `apps/clinic-portal/src/app/ClinicIntercomProvider.tsx`, env `NEXT_PUBLIC_INTERCOM_APP_ID`.

### 2.3 Admin Next.js — `apps/admin`

- **Framework:** Next.js 15 (App Router), port 3002 in dev.
- **Deploy:** **Cloudflare Pages (static export).** Workflow `deploy-admin-staging.yml` (develop → staging only — no production-admin workflow inspected at HEAD). Triggers on `apps/admin/**` + `packages/{ui,env,config,observability}/**`.
- **Observability gap (D-known):** **No Sentry integration in admin** — errors invisible in production (CLAUDE.md § 8 + `openspec/specs/admin/spec.md` § Known gaps, flagged as C1 MEDIUM).
- Routes are internally rewritten from `/admin/*` per the auth spec REQ.

### 2.4 NestJS API — `apps/api` (`@dentour/api`, package version `0.12.1`)

- **Framework:** NestJS 10 (`@nestjs/{common,core,platform-express,config,schedule,swagger,throttler,cache-manager} ^10`), port 4000 in dev, global prefix `/api/v1`. Swagger at `localhost:4000/docs`.
- **ORM:** Prisma `~5.22.0` (`@prisma/client` matches `packages/database` schema).
- **Auth:** custom Supabase JWT verifier — Bearer-token middleware for mutations; cookie-based session accepted for reads (per `auth` capability spec).
- **Security middleware:** `helmet ^7.1`, `cookie-parser ^1.4`, `csrf-csrf ^4.0` (CSRF library is *present in dependencies* but the audit's D3 finding says "No CSRF protection" — implies the middleware is scaffolded but disabled in current routes, with the explicit per-route skip on `/api/v1/clinic-onboarding/*` documented in the `clinic-onboarding` spec). Webhooks via `svix ^1.84`.
- **Caching:** `@nestjs/cache-manager`, `cache-manager-ioredis-yet`, `ioredis ^5.9` → **Redis caching layer**, not surfaced in CLAUDE.md or proposal. **Flag for kickoff** — Redis is an integration the topology should remember.
- **Observability:** `@sentry/nestjs ^10.36`, `@sentry/profiling-node ^10.36`. Sourcemaps uploaded by `deploy-railway.yml` to `dentour` org, `api-nestjs` project.
- **Validation:** `class-validator ^0.14` + `class-transformer ^0.5` + Zod **4** (`zod ^4.3.5`). All other packages use Zod 3. CLAUDE.md § 5 / § 9 D6 explicitly forbids sharing Zod schemas across the API/everything-else boundary.
- **Deploy:** Railway via `deploy-railway.yml`. Develop branch auto-deploys to staging (`api-staging.dentour.eu`); main is gated on the CI workflow completing successfully (`workflow_run` trigger) before auto-deploying to production (`api.dentour.eu`). Railway CLI v3/v4 (the workflow tries `railway up`, falls back to a direct GraphQL `serviceInstanceRedeploy` mutation if CLI fails).

### 2.5 Fumadocs documentation — `apps/docs`

- **Framework:** Next.js 15 + Fumadocs (per CLAUDE.md § 2). Port 3004 in dev.
- **Deploy:** Cloudflare Pages via `deploy-docs-staging.yml` and `deploy-docs-production.yml`. Static export.
- **Purpose:** internal/external documentation portal — not customer-facing booking surface.

### 2.6 E2E test runner — `apps/e2e-tests` (`@dentour/e2e-tests`)

- **Framework:** Cypress (per CLAUDE.md § 7 — "real environments only, no mocks"). Spec organisation: `cypress/e2e/{clinic,patient,admin,smoke,accessibility}/**/*.cy.ts`.
- **Triggered by:** `ci.yml` `e2e_patient_onboarding` and `e2e_clinic_onboarding` jobs (against `https://staging.dentour.eu` / `https://clinic-staging.dentour.eu`); also `e2e-staging.yml` and `e2e-monitoring.yml` workflows. **E2E onboarding failures block merges to main** (DEN-144 referenced in `ci.yml` § Job 6a/6b comments).
- Custom commands in `cypress/support/commands/`. Test data prefixed `e2e_`.

### 2.7 MCP server — `apps/mcp-server` (`@dentour/mcp-server`)

- **Framework:** Node.js with the Model Context Protocol stdio runtime (per CLAUDE.md § Glossary entry MCP).
- **Build:** `pnpm turbo run build --filter=@dentour/mcp-server` depends on `@dentour/database#db:generate`. Output `dist/index.js`. Started via `infisical run --env=dev --path=/shared -- node apps/mcp-server/dist/index.js --stdio`.
- **Deploy:** none (developer-tool only; never network-deployed).

### 2.8 AI-team CLI — `apps/ai-team` (`@dentour/ai-team`)

- **Framework:** Node.js with `@anthropic-ai/sdk ^0.30`, `commander ^12`, `xstate ^5.18`. Bin `dentour-ai`.
- **Internal deps:** zero — operates standalone against Anthropic + filesystem + `simple-git`. This is Dentour's own agent runtime, not part of the platform request path.
- **Deploy:** none (CLI-only).

### 2.9 Vestigial — `apps/auth-portal`

- Contains only a `tsconfig.tsbuildinfo` file. No `package.json`. Auth logic lives in `@dentour/auth`. CLAUDE.md § 2 footnote confirms "vestigial — do not modify". Phase 6 convention adapter MUST NOT propose work in this folder.

### 2.10 Storybook (UI package, not an app)

- **Location:** lives **inside** `packages/ui` (`pnpm storybook` in `packages/ui/package.json`), not as a sibling app. Deployed via `deploy-storybook.yml` as a Cloudflare Pages project (`dentour-ui-storybook`). Visual-regression baselines.

---

## 3. Packages inventory

Twenty internal packages: **eighteen top-level** under `packages/` plus **two feature sub-packages** under `packages/features/`. All packages are `private: true` with `version: "0.x.y"`. Build tooling varies — `tsup` for libraries that emit `dist/`; `@dentour/auth` and `@dentour/database` build differently (see below).

| Package | Build output | Public API summary |
|---|---|---|
| `@dentour/ui` | `dist/index.{js,cjs,d.ts}` + `./styles`, `./fonts`, `./tokens` (tsup + tailwind) | shadcn/ui + Radix primitives + 550+ exports incl. domain components (`DentalChart`, `PriceEditor`, `CountrySelect`, `ClinicAvatar`). Heavy Radix surface (`@radix-ui/react-{accordion, alert-dialog, avatar, checkbox, dialog, dropdown-menu, label, navigation-menu, popover, progress, radio-group, scroll-area, select, separator, slider, slot, switch, tabs, toast, tooltip}`). Imports `@dentour/icons`, `@dentour/query-keys`, `@dentour/types`. Co-hosts a Storybook 10 instance. |
| `@dentour/auth` | `src/` direct (no `dist/` — points `main`/`types` at `./src/index.ts`) | Supabase Auth singleton (`getSupabaseClient`), `AuthProvider`, `RoleGuard`, `useAuth`. Subpath exports `./client`, `./components`, `./types`. Imports `@dentour/observability`. Peer-deps `@dentour/ui`, `next ^14 || ^15`, `react ^18 || ^19`. |
| `@dentour/database` | `dist/` (tsup) + `prisma/schema.prisma` | Prisma client wrapper (`PrismaService`), generated `types.ts`, `@map(snake_case)` columns, branded UUIDs, `priceCents Int`. Per-package `db:generate` task that turbo runs as a pipeline prerequisite for API + messaging. |
| `@dentour/api-client` | `dist/` (tsup) + generated `src/generated/` | Auto-generated Axios client from Swagger spec. **CLAUDE.md § 5 forbids hand-editing.** Per CLAUDE.md § 9 finding A1 — **currently consumed by no app** (each frontend has its own fetch wrapper); generation pipeline exists, integration deferred. |
| `@dentour/types` | `dist/` (tsup) | Shared TypeScript types; `Money` helpers; `formatPriceCents` (used by `multi-currency` capability). |
| `@dentour/routes` | (per `turbo.json` config: `build` task implicit) | Type-safe route definitions (`PATIENT_DASHBOARD_ROUTES`, `CLINIC_DASHBOARD_ROUTES`), `buildPath('/:locale/clinics/:slug', 'en', { slug })`. Source-of-truth for the 7-domain registry in `packages/routes/src/constants/domains.ts` (referenced by `multi-domain-routing` spec). |
| `@dentour/env` | `dist/` (tsup) | Per-app Zod 3 schemas (`env.web`, `env.api`, `env.admin`, `env.clinicPortal`); `SKIP_ENV_VALIDATION` honored everywhere. |
| `@dentour/i18n` | (lib) | Shared **domain** translations (services / categories / specialties / pricing) — separate from `next-intl` app-level translations which live in `apps/<app>/messages/`. |
| `@dentour/query-keys` | `dist/` (tsup) | TanStack Query 5 key factories (`clinicKeys`, `quoteKeys`, `userKeys`, …). |
| `@dentour/config` | (lib) | Multi-domain configuration; per-domain feature flags (e.g. `.co.il` enables Hebrew features). Domain registry under `packages/config/src/domains/dentour-*.ts`. Imports nothing internal. |
| `@dentour/features/patient-onboarding` | `dist/` (tsup) | XState v5 state machine for patient onboarding wizard. Consumed by `apps/patient-web`. Imports `@dentour/database`, `@dentour/env` (indirect via `apps/api`). |
| `@dentour/features/clinic-onboarding` | `dist/` (tsup) | XState v5 state machine for clinic onboarding wizard. Consumed by `apps/clinic-portal`. Includes a photo-upload service helper. |
| `@dentour/maps` | `dist/` (tsup) | Google Maps wrapper (`@googlemaps/js-api-loader ^1.16`, `@googlemaps/markerclusterer ^2.5`). React peer-dep. |
| `@dentour/messaging` | `dist/` (tsup) | **Multi-channel messaging.** Direct `resend ^4.0` dependency for transactional email; `@react-email/components ^0.0.25` for templates. Subpath exports `./types`, `./channels`, `./intercom`. **Intercom is a runtime subpath** (no `intercom-client` npm dep — likely embeds Intercom JS client lib or uses HTTP directly). |
| `@dentour/analytics` | `dist/` (tsup) | GA4 wrapper. Consumes 7 per-domain measurement IDs (`NEXT_PUBLIC_GA_ID_{EU,IL,MD,RO,GE,ES,UK}`) injected at build time. Peer-deps `next`, `react`. Imports `@dentour/ui`. |
| `@dentour/observability` | `dist/{shared,server,client,next}/index.{js,cjs,d.ts}` (tsup) | Four subpath exports for the four runtime contexts. **No runtime npm deps** — pure infra utility. `scrubSensitiveData()`, `createServerLogger()`, `createApiClient()`. Per CLAUDE.md § 5: this is the only loggers permitted in production (no `console.log`). |
| `@dentour/utils` | (lib) | Generic helpers. |
| `@dentour/icons` | (lib) | Custom dental SVG icon components, exported from Figma. |
| `@dentour/design-tokens` | (lib) | Design token exports. **CLAUDE.md § 2 marks this "may be unused — verify before depending on it"** — flag for the drift audit. |
| `@dentour/cli` | (lib) | Internal CLI for Cloudflare infrastructure management. Not surfaced in any documented end-user workflow. |

**Tooling (`tooling/`):**

- `tooling/tailwind-config` — shared Tailwind CSS configuration. The OKLCH gotcha in CLAUDE.md § 10 documents that apps must extend this *and* define hover/pressed CSS vars in their own `globals.css`; consuming this config alone is not enough.
- `tooling/typescript-config` — base, Next.js, and react-library tsconfig presets.

### 3.1 Internal dependency graph

The graph is shallow with two notable invariants: (1) the database/types primitives have no internal deps; (2) `@dentour/auth` and `@dentour/ui` are the two convergence points everything else funnels into:

```
                  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
                  │ @dentour/types   │    │ @dentour/utils   │    │ @dentour/config  │
                  │  (zero internal) │    │  (zero internal) │    │  (zero internal) │
                  └────────┬─────────┘    └──────────────────┘    └──────────────────┘
                           │
                           ▼
                  ┌──────────────────┐    ┌──────────────────┐
                  │ @dentour/        │    │ @dentour/        │
                  │ database         │    │ env (Zod 3)      │
                  │ (Prisma)         │    └─────────┬────────┘
                  └────────┬─────────┘              │
                           │                        │
            ┌──────────────┼─────────────┐          │
            ▼              ▼             ▼          │
    ┌─────────────┐  ┌─────────────┐ ┌────────────┐ │
    │ @dentour/   │  │ @dentour/   │ │ @dentour/  │ │
    │ messaging   │  │ features/   │ │ features/  │ │
    │ (Resend)    │  │ patient-    │ │ clinic-    │ │
    └──────┬──────┘  │ onboarding  │ │ onboarding │ │
           │         └──────┬──────┘ └─────┬──────┘ │
           │                │              │        │
           │                │              │        ▼
           │                │              │  ┌──────────────┐
           │                │              │  │ apps/api     │
           │                │              │  │ (NestJS,     │
           │                │              │  │ Zod 4, Prisma│
           │                │              │  │  + Redis)    │
           │                │              │  └──────────────┘
           ▼                ▼              ▼
    ┌──────────────────────────────────────────┐
    │ apps/patient-web / apps/clinic-portal    │  ← also import @dentour/ui,
    │ (Next.js 15)                             │     @dentour/auth, @dentour/i18n,
    └──────────────────────────────────────────┘     @dentour/maps, @dentour/analytics,
                                                     @dentour/query-keys, @dentour/routes,
                                                     @dentour/icons, @dentour/observability

    ┌──────────────────┐
    │ @dentour/        │  consumes:
    │ observability    │   @dentour/auth (peer, runtime)
    │ (zero deps)      │   apps/api (server subpath)
    └──────────────────┘   apps/*-web (client / next subpath)

    ┌──────────────────┐
    │ @dentour/ui      │  consumes: @dentour/icons, @dentour/query-keys, @dentour/types
    │ (Radix +         │  consumed by: every Next.js app + @dentour/auth (peer)
    │  shadcn/ui)      │              + @dentour/analytics
    └──────────────────┘

    ┌──────────────────┐
    │ @dentour/        │  consumes: @dentour/ui, @dentour/database, @dentour/maps,
    │ ai-team          │  observability — **but its source has zero workspace deps**
    │ (Anthropic SDK)  │  (operates as standalone CLI; CLAUDE.md lists it as Agent runtime)
    └──────────────────┘
```

Key invariants observed:

- **`apps/web ↔ apps/clinic-portal ↔ apps/admin` never import each other.** Cross-app communication is exclusively via the NestJS API.
- **`@dentour/api-client` is the *intended* shared HTTP layer but is consumed by no app at HEAD.** Each app has its own `lib/api/*` fetch wrapper (CLAUDE.md § 9 A1 — MEDIUM, acknowledged). The convention adapter must understand this divergence: any spec change that references the api-client must verify whether it lands in the generated package or in each app's per-app wrapper.
- **`@dentour/ai-team` does not import workspace packages.** It is structurally separate from the platform request path and is shipped only as a Node CLI.
- **Zod 3/4 boundary lives at `apps/api`.** `apps/api/package.json` declares `zod ^4.3.5`; everything else uses `zod ^3.23.8`. Per CLAUDE.md § 5 / § 9 D6 — schemas MUST NOT cross this boundary.

---

## 4. Entry points and runtime topology

| Surface | Entry file | Runtime | Triggered by |
|---|---|---|---|
| `patient-web` SSR/RSC (Pages) | `apps/patient-web/src/app/[locale]/layout.tsx` + middleware | Cloudflare Pages (static export, no server runtime) | HTTP traffic via Cloudflare CDN; per-region project mapped to per-region custom domain. |
| `patient-web` SSR (Workers) | `apps/patient-web/.open-next/worker.js` | Workers + OpenNext | `pnpm opennext:deploy:staging` (CI: `deploy-staging-ssr.yml`). Production path is **not** wired. |
| `clinic-portal` | `apps/clinic-portal/src/app/[locale]/layout.tsx` | Cloudflare Pages (static export) | HTTP traffic via Cloudflare CDN. |
| `admin` | `apps/admin/src/app/[locale]/layout.tsx` | Cloudflare Pages (static export) | HTTP traffic via Cloudflare CDN. |
| `api` | `apps/api/src/main.ts` (bootstrap NestJS) | Node 20+ (Railway container) | HTTP traffic on `api.dentour.eu` (prod) / `api-staging.dentour.eu` (staging). Global prefix `/api/v1`. |
| `docs` (Fumadocs) | `apps/docs/src/app/...` | Cloudflare Pages (static export) | HTTP via `docs.dentour.*` (subdomain not surfaced in deploy YAMLs inspected; flag for kickoff). |
| `mcp-server` | `apps/mcp-server/dist/index.js --stdio` | Node 20 (developer-local) | `pnpm mcp:start`. Never network-deployed. |
| `ai-team` | `apps/ai-team/dist/index.js` (`bin: dentour-ai`) | Node 20 (developer-local) | `dentour-ai <subcommand>`. Never network-deployed. |
| `e2e-tests` | `apps/e2e-tests/cypress.config.ts` | Node 20 + Chromium | `pnpm e2e:open` / `pnpm e2e:staging` / CI workflows (`ci.yml`, `e2e-monitoring.yml`, `e2e-staging.yml`). |
| Storybook | `packages/ui/.storybook/main.ts` (lives inside `@dentour/ui`) | Static (Cloudflare Pages) | `pnpm storybook` (dev), `pnpm --filter @dentour/ui build-storybook` (output `storybook-static/`). |

---

## 5. Deploy targets

Dentour's deploy surface uses **three cloud providers**, all controlled from GitHub Actions:

### 5.1 Cloudflare Pages — patient-web, clinic-portal, admin, docs, ui-storybook

- **Patient-web:** per-region projects (`dentour-production-{eu,il,md,ro,ge,es,uk}` for prod, `dentour-patient-web-staging-{eu,il,md,ro,ge,es,uk}` for staging). 7-region matrix; production gated `max-parallel: 1`.
- **Clinic-portal / admin / docs:** single per-environment Cloudflare Pages project (no per-region matrix — these apps are not multi-locale-per-domain).
- **Static export across the board.** `STATIC_EXPORT: true` in build env; `wrangler pages deploy <out-dir>` step. No SSR on Pages.
- **Trigger:** push to `main` → production; push to `develop` → staging. Pull-request previews wired for `clinic-portal` (per `deploy-preview.yml` and `deploy-clinic-portal.yml` pull_request triggers).

### 5.2 Cloudflare Workers — patient-web (SSR, staging-only)

- `deploy-staging-ssr.yml` runs `pnpm --filter=patient-web... build` + `wrangler deploy --config wrangler.workers.toml --env staging` for the 7-region matrix on every push to `develop`.
- **No equivalent production-SSR workflow exists at HEAD.** This makes the SSR path effectively a staging-only experiment despite the governance docs claiming it is the production target.

### 5.3 Railway — NestJS API + (potentially) other services

- `deploy-railway.yml` deploys `apps/api`. Two service IDs are referenced via secrets: `RAILWAY_SERVICE_ID_STAGING` (auto-deploys on push to develop) and `RAILWAY_SERVICE_ID_PROD` (auto-deploys on successful CI workflow_run against main).
- Build runs the entire `pnpm turbo run build --filter=@dentour/api...` graph inside the GitHub Actions runner; Railway CLI v3.20.0 deploys the result. Health probe against `/api/v1/health` with 5-minute timeout.
- Sentry sourcemaps uploaded to `api-nestjs` project after deploy.

### 5.4 Supabase — `pnpm db:seed` / migrations

- Database is Supabase PostgreSQL. RLS posture not directly inspected (will be Phase 2 drift-audit material against the auth + multi-domain-routing specs).
- Migrations under `packages/database/prisma/migrations/`; applied via `cd packages/database && pnpm db:migrate` (per CLAUDE.md § 3, this script is **not** at the root). Seed scripts (`pnpm db:seed:full`, `:staging`, `:e2e`) are root-level and wrapped in `infisical run`.
- Supabase Auth handles phone OTP, magic link, cookie sessions. Supabase Storage hosts the `patient-xrays` and `clinic-photos` buckets (referenced by `patient-onboarding` and `clinic-onboarding` specs respectively).

### 5.5 Infisical — secrets vault

- **Vault:** Infisical (organisation "Dentour"). Path structure per CLAUDE.md § 4: `/shared`, `/web`, `/api`, `/admin`, `/clinic-portal`, `/auth-portal`. The `/auth-portal` path persists despite the app being vestigial.
- **CI:** Infisical CLI is *not* surfaced in the deploy workflows (which use GitHub Secrets directly). Local dev wraps `pnpm dev` in `infisical run --env=dev --path=/shared`. `pnpm env:pull:all` exports to `.env.local` for offline work.

---

## 6. CI / CD inventory

`.github/workflows/` contains **17 files**:

| Workflow | Trigger | Purpose / gate |
|---|---|---|
| `ci.yml` | PR + push to main/develop | Format / Lint / Type-check / API tests (Jest + cov) / Build / Lighthouse a11y (non-blocking) / Dependency review (non-blocking) / Validate Claude Skills / **E2E patient onboarding (blocking)** / **E2E clinic onboarding (blocking)** / Summary. 9 jobs; ~567 lines. |
| `deploy-staging.yml` | Push to `develop` (paths-filtered to web/packages) | Patient-web → Cloudflare Pages (7-region matrix); Sentry sourcemaps. |
| `deploy-staging-ssr.yml` | Push to `develop` (paths-filtered to web/packages) | Patient-web → Cloudflare Workers (7-region matrix). Staging-only. |
| `deploy-production.yml` | Push to `main` + manual `workflow_dispatch` (gated "type 'deploy' to confirm") | Patient-web → Cloudflare Pages (7-region matrix). `max-parallel: 1`. Health-curl verification + GitHub Deployments API entry per region. |
| `deploy-clinic-portal.yml` | Push/PR to main+develop (paths-filtered) + manual dispatch | Clinic-portal → Cloudflare Pages. |
| `deploy-admin-staging.yml` | Push to `develop` (paths-filtered) + manual dispatch | Admin → Cloudflare Pages (staging). |
| `deploy-docs-staging.yml` | (paths-filtered to docs) | Docs site → Cloudflare Pages (staging). |
| `deploy-docs-production.yml` | (paths-filtered to docs) | Docs site → Cloudflare Pages (production). |
| `deploy-preview.yml` | PR | Preview deploys (likely PR-scoped Cloudflare Pages previews). |
| `deploy-railway.yml` | Push to `develop` (staging) + `workflow_run` after CI passes on `main` (production) + manual dispatch | API → Railway. Health-probe `/api/v1/health` with 5-minute timeout. Sentry sourcemaps after success. |
| `deploy-storybook.yml` | (UI changes) | Storybook → Cloudflare Pages (`dentour-ui-storybook`). |
| `e2e-monitoring.yml` | Schedule + manual | Synthetic Cypress runs against production-like staging URLs. |
| `e2e-staging.yml` | Manual / scheduled | Extended Cypress runs against staging. |
| `health-monitor.yml` | Schedule | External health-monitor calls (likely uptime + `/api/v1/health`). |
| `version-and-release.yml` | (on push to main) | Changesets release flow — creates "Version Packages" PR; on merge, publishes packages to NPM. |
| `version-check.yml` | PR | Asserts that PRs touching publishable packages include a changeset. |
| `.github/workflows/README.md` | docs | Inventory page (counted as a file but is not itself an executable workflow). |

Notable absences and gotchas for the convention adapter:

- **No workflow runs `pnpm openspec:validate`.** **This contradicts [`onboard-dentour/design.md`](../../changes/onboard-dentour/design.md) § Why Dentour is not a copy-paste of Unbind item 2 ("Dentour's CI requires it to pass").** Reality: validation is a local-only step invoked manually by `/opsx:apply` and `/opsx:archive`. The validate command is *defined* (`pnpm openspec:validate` → `openspec validate --all --strict`) but it is **not gated by any CI workflow** at HEAD. Either the convention-adapter design (Phase 6) wires it in as the studio's contribution to Dentour's CI, or the studio must operate without machine-checkable validation on the contributor side.
- **Skill-validation gate exists.** `ci.yml` § Job 8 `validate-skills` runs `node scripts/ci/validate-skills.mjs` and grep-checks for accidentally-committed secrets in any `.claude/skills/*/SKILL.md`. The convention adapter must NOT generate SKILL.md files in changes — or if it does, must satisfy this gate.
- **E2E onboarding is a merge gate.** Per `ci.yml` § Job 6a/6b comments (DEN-144), patient + clinic onboarding E2Es block merging to main. Any change to those flows must run against the staging environment in CI; the studio bot in Hybrid mode targets a `studio/<slug>` branch — confirm at kickoff that the human-managed merge to main will still trigger this gate.
- **Commit scope warning.** `commitlint.config.js` `scope-enum` is severity 2 (error). Studio commits scoped `(onboard-dentour)` will **hard-fail commitlint** — not warn-but-pass like Vivod. Either register a new scope (e.g., `forge`, `onboard-dentour`, or a generic `studio`) or land via an allowed scope (`docs`, `workspace`, or `features`).

---

## 7. OpenSpec corpus

### 7.1 Layout

Dentour uses `openspec/specs/<capability>/spec.md` as the source-of-truth tree. Twelve capability folders, one per capability. **There is no `openspec/current/` folder, no `INDEX.md` ordering file beyond the human-facing one, and no `openspec/AGENTS.md`** — the tasks.md "if exists" hedge resolves to "does not exist". Authoring guidance lives in `openspec/config.yaml` instead.

```
openspec/
├── INDEX.md                              # Human-facing capability index
├── config.yaml                           # Project context + authoring rules
├── specs/
│   ├── auth/spec.md                      # Foundations — stable
│   ├── multi-domain-routing/spec.md      # Foundations — stable
│   ├── multi-currency/spec.md            # Foundations — stable
│   ├── patient-onboarding/spec.md        # Domain core — stable
│   ├── clinic-onboarding/spec.md         # Domain core — stable
│   ├── quote-workflow/spec.md            # Domain core — stable
│   ├── clinic-profile/spec.md            # Domain core — stable
│   ├── patient-account/spec.md           # Surface — Phase 0 skeleton ⚠️
│   ├── clinic-portal/spec.md             # Surface — Phase 0 skeleton ⚠️
│   ├── admin/spec.md                     # Surface — Phase 0 skeleton ⚠️
│   ├── dental-chart/spec.md              # Surface — Phase 0 skeleton ⚠️
│   └── messaging/spec.md                 # Surface — Phase 0 skeleton ⚠️ (deprecating)
└── changes/
    └── archive/                           # Currently empty subtree at HEAD;
                                           # no active changes
```

### 7.2 Module spec inventory + Phase 0 / Phase 1 status

The proposal/design assumed all 12 capabilities are fully specified. **Reality: only 7 are stable; the remaining 5 are explicit Phase-0 skeletons containing only a `### Requirement: skeleton — to be filled in Phase 1` placeholder.** Phase 1 retrospective baseline authoring is Dentour's own backlog, scoped as "Wave C" per the skeleton specs' `## Next` footer.

| # | Capability | Status | Requirement style | Notes |
|---|---|---|---|---|
| 1 | `auth` | **stable** | `### Requirement: <statement>` + `#### Scenario:` WHEN/THEN | 4 requirements, mentions `D3` (no CSRF) and `D7` (CORS pattern) as known gaps. |
| 2 | `multi-domain-routing` | **stable** | same | 5 requirements, 7-domain registry confirmed, admin filter UI lags backend (only EU/IL/MD listed). |
| 3 | `multi-currency` | **stable** | same | 6 requirements, ECB cron, stale-rate handling, cold-start fallback dated 2026-03-10. |
| 4 | `patient-onboarding` | **stable** | same | 5 requirements + scenarios for 3 paths (path1/path2/pathB), bearer-token-aware submit (PR #317). |
| 5 | `clinic-onboarding` | **stable** | same | 6 requirements, **explicit CSRF skip on `/api/v1/clinic-onboarding/*`** documented as REQ. |
| 6 | `quote-workflow` | **stable** | same | 7 requirements, 5-state FSM enforced in service layer (not DB), currency fixed at quote creation. |
| 7 | `clinic-profile` | **stable** | same | 6 requirements, hreflang alternates, returnUrl open-redirect prevention (PR #316). |
| 8 | `patient-account` | **skeleton (Phase 0)** | placeholder only | Phase 1 will document dashboard surfaces + TODO stubs. |
| 9 | `clinic-portal` | **skeleton (Phase 0)** | placeholder only | Phase 1 will document portal surfaces + role/permission model. |
| 10 | `admin` | **skeleton (Phase 0)** | placeholder only | Notes C1 (no Sentry) + D7 (CORS) as known gaps; Phase 1 will document moderation workflow. |
| 11 | `dental-chart` | **skeleton (Phase 0)** | placeholder only | Phase 1 will document tooth-numbering convention + treatment-editor → quote-line-item mapping. |
| 12 | `messaging` | **skeleton (Phase 0), deprecating** | placeholder only | Explicit super-session marker: "to be partially superseded by AI Communication Layer (Phase 2) once `add-patient-clinic-messaging-relay` lands". |

### 7.3 Convention specifics for the Forge adapter (Phase 6 input)

- **Heading style is uniform across stable specs.** `### Requirement: <statement>` (full statement in the heading, not `REQ-XXX-NNN`-style IDs). Scenario blocks are `#### Scenario: <name>` followed by `- **WHEN** … / - **THEN** …` bullets. `clinic-profile` and `quote-workflow` use a `- **GIVEN** …` preamble in a few scenarios.
- **`## Purpose` is mandatory** at the top of every spec (skeletons included).
- **`## Implementation map` is mandatory** per `openspec/config.yaml § rules.spec` and present in every stable spec — pointing to `apps/<app>/src/...` paths and `packages/<pkg>/...` source files. The design.md assumption "Implementation map points to `apps/`, `packages/`" is correct.
- **`## Known gaps / Acknowledged risks`** is the canonical name for the risks section per `openspec/config.yaml`. Three stable specs use this; two use `## Status` for skeleton lifecycle metadata. Phase 6 parser must accept both.
- **Deltas live at `openspec/changes/<slug>/specs/<cap>/spec.md`** per the formal-deltas convention. **However, the `openspec/changes/` directory is currently empty at HEAD** (only `archive/` subfolder, itself empty of contents in this scan); no live deltas to inspect. The convention is documented in CLAUDE.md § 0 and `openspec/config.yaml` § rules.proposal+tasks but unused at HEAD. The `add-product-charter`, `add-realtime-sync-and-route-cache`, etc. example shapes that exist in Vivod do not have Dentour equivalents at this commit.
- **`pnpm openspec:list`** is intended to surface specs and requirement counts; this is the tool the convention adapter should run as a sanity check before emitting a delta.

### 7.4 Active changes at HEAD

None. `openspec/changes/` contains only an empty `archive/` directory. This means Dentour is between change-cycles at the anchor commit (the HEAD commit itself bootstrapped the workflow on 2026-04-30; no proposal has been opened since). **The studio's first Forge run on Dentour will be the first delta authored by anyone since the workflow shipped.** Phase 6 of `onboard-dentour` must not assume there is a worked-example proposal in Dentour's tree to mirror.

---

## 8. External integrations

| Provider | Use | Wired via | Secret name(s) (per Infisical paths + workflow envs) |
|---|---|---|---|
| **Supabase** | Postgres + Auth (phone OTP / magic link / cookie sessions) + Storage (`patient-xrays`, `clinic-photos` buckets) | `@supabase/supabase-js ^2.86` (everywhere), `@supabase/ssr ^0.8` (web), `apps/api` service-role client | `DATABASE_URL`, `DIRECT_URL`, `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, `SUPABASE_JWT_SECRET`, `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `*_PROD` variants for production |
| **Resend** | Transactional email (quote response, status change, verification) | `resend ^4.0.0` in `@dentour/messaging`; `@react-email/components ^0.0.25` for templates | `RESEND_*` (per Infisical `/api` path in CLAUDE.md § 4) |
| **Intercom** | Patient↔clinic in-app chat, client-side widget | `apps/patient-web/src/components/providers/IntercomProvider.tsx` + `apps/clinic-portal/src/app/ClinicIntercomProvider.tsx`; `@dentour/messaging/intercom` subpath | `NEXT_PUBLIC_INTERCOM_APP_ID` (client-side; injected at build time in `deploy-{staging,production}.yml`) |
| **Sentry** | Error tracking + sourcemaps for `patient-web` (Sentry project `patient-web`) and `apps/api` (Sentry project `api-nestjs`). Also wired in `clinic-portal`. **Not wired in admin (CLAUDE.md § 8, finding C1).** | `@sentry/nextjs ^10` (web + clinic-portal), `@sentry/nestjs ^10.36` + `@sentry/profiling-node ^10.36` (API) | `NEXT_PUBLIC_SENTRY_DSN`, `SENTRY_DSN_PATIENT_WEB`, `SENTRY_AUTH_TOKEN`, `SENTRY_ORG: dentour`, `SENTRY_PROJECT: patient-web` / `api-nestjs` |
| **Google Maps** | Clinic location pin + autocomplete in onboarding | `@googlemaps/js-api-loader ^1.16`, `@googlemaps/markerclusterer ^2.5` in `@dentour/maps` | `NEXT_PUBLIC_GOOGLE_MAPS_KEY` |
| **GA4 (Google Analytics)** | Web analytics; **per-domain** measurement IDs (one per region) | `@dentour/analytics` package | `NEXT_PUBLIC_GA_ID_EU`, `NEXT_PUBLIC_GA_ID_IL`, `NEXT_PUBLIC_GA_ID_MD`, `NEXT_PUBLIC_GA_ID_RO`, `NEXT_PUBLIC_GA_ID_GE`, `NEXT_PUBLIC_GA_ID_ES`, `NEXT_PUBLIC_GA_ID_UK` (7 IDs) |
| **Anthropic** | Internal AI workflows: `apps/ai-team` (Anthropic SDK) + `.claude/skills/*` agent-runtime tooling. **Not in the public booking request path.** | `@anthropic-ai/sdk ^0.30` in `apps/ai-team`; Claude Code via `.claude/skills/dentour-openspec-workflow/SKILL.md` | `ANTHROPIC_API_KEY` (developer-local; not in production secrets per workflow inspection) |
| **Cloudflare** | Pages hosting (every Next.js app), DNS for 7 domains, Workers (SSR staging path), TLS | `wrangler@3.90.0` (deploy-staging) / `wrangler@4.64.0` (deploy-production root); `@opennextjs/cloudflare ^1.16.4` (patient-web SSR path) | `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID` |
| **Railway** | API hosting (`api.dentour.eu` / `api-staging.dentour.eu`) | `@railway/cli@3.20.0` invoked from `deploy-railway.yml`; fallback to GraphQL `serviceInstanceRedeploy` mutation | `RAILWAY_TOKEN` (project token), `RAILWAY_PROJECT_ID`, `RAILWAY_SERVICE_ID_PROD`, `RAILWAY_SERVICE_ID_STAGING` |
| **Infisical** | Centralized secret vault (organisation "Dentour"). Env-scoped (dev/staging/prod) with path namespaces (`/shared`, `/web`, `/api`, `/admin`, `/clinic-portal`, `/auth-portal`). | `infisical` CLI wrapped around `pnpm dev` / per-app dev scripts; **not surfaced in deploy workflows** (which use GitHub Secrets directly). `pnpm env:pull:all` for offline export. | `INFISICAL_*` (developer-side login) |
| **Redis** | API caching layer for the NestJS app | `cache-manager-ioredis-yet ^2.1`, `ioredis ^5.9` in `apps/api` | (Redis URL in API env — not exposed in workflow inspection) |
| **Linear** | Issue tracking; issue prefix `DEN-` (per `openspec/config.yaml`); used internally by Dentour. `apps/ai-team` does not import the Linear SDK at HEAD. | `@linear/sdk ^65.2` is a root devDependency (likely used by scripts/tooling, not runtime) | (Linear API token — not surfaced in deploy workflows) |
| **GitHub** | Code hosting (`dentour-org/dentour`), Actions CI/CD, Release-Please / Changesets-driven NPM publish | 17 workflows; `@changesets/changelog-github ^0.5` | `GITHUB_TOKEN` (default), per-deploy `secrets.GITHUB_TOKEN` for Releases API in `deploy-production.yml` |
| **Turborepo Remote Cache** | Build cache | `TURBO_TOKEN` + `TURBO_TEAM` env in every workflow | `TURBO_TOKEN`, `TURBO_TEAM` (Vercel free-tier per `docs/06-development/turborepo-remote-cache.md` referenced in CLAUDE.md) |
| **Figma** | Design source + Code Connect | `@figma/code-connect ^1.3` root devDep; `figma.config.json` at root | (Figma personal-access-token — developer-local) |

**Implicit / scaffolded but not yet wired:**

- **Stripe.** The Infisical path structure in CLAUDE.md § 4 lists `/api` containing `STRIPE_*`. `apps/api/src/env.ts` and `packages/env/src/schemas.ts` reference Stripe env-var names, and `apps/patient-web/env.template` includes Stripe placeholders. **No `stripe` npm dependency exists in any `package.json` at HEAD, and no `apps/api/src/**/*stripe*` source files were found in `apps/api/src/`.** Payments are **env-scaffolded but not yet implemented** in the request path. The `onboard-dentour` proposal § Why and design.md § 3 list Stripe alongside Resend / Sentry / Intercom / Google Maps — this conflates intent with implementation. **Flag.**
- **Twilio.** Not directly referenced; SMS for phone OTP (per `auth` capability) is delegated to Supabase Auth's SMS provider, which is configurable at the Supabase dashboard level rather than via Dentour npm deps.

---

## 9. Studio-side observations and open flags

These are **studio-internal notes** for downstream phases of `onboard-dentour` — not assertions about Dentour, just things to verify or remember:

1. **Production patient-web is on Cloudflare Pages (static export), not Cloudflare Workers SSR.** CLAUDE.md § 1, `openspec/config.yaml` § context, and `.github/copilot-instructions.md` § Overview all describe patient-web as "Cloudflare Workers (SSR via OpenNext)". `deploy-production.yml` and `deploy-staging.yml` both set `STATIC_EXPORT: "true"` and run `wrangler pages deploy`. The SSR/Workers path (`deploy-staging-ssr.yml` + `next.config.ssr.js` + `opennextjs-cloudflare`) is **wired but staging-only**. This is the largest single drift in this scan. Resolve at kickoff or in Phase 2 drift audit.
2. **Locale catalog ships 6 locales, governance says 4.** `apps/patient-web/messages/` contains `en, es, he, ka, ro, ru`. CLAUDE.md / config.yaml / copilot-instructions all describe `en | ru | ro | he` only. Spanish (`es`) and Georgian (`ka`) were added for `dentour.es` / `dentour.ge` but the governance docs were not updated.
3. **Five of 12 capability specs are Phase-0 skeletons.** `patient-account`, `clinic-portal`, `admin`, `dental-chart`, and `messaging` contain only a `### Requirement: skeleton — to be filled in Phase 1` placeholder. The drift-audit phase (T2.x) is dramatically lighter-weight for these five — there is no spec content to drift against; the audit reduces to "code is undocumented" findings. The convention adapter design (T6.1) should still parse all 12, but Phase 2 deliverable shape will be 7 substantive rows + 5 "skeleton → undocumented code" rows.
4. **`pnpm openspec:validate` is NOT in CI.** `onboard-dentour/design.md` § Why Dentour is not a copy-paste of Unbind item 2 asserts "Dentour's CI requires it to pass". Reality: `ci.yml` does not invoke it; `pnpm openspec:validate` is a local-only step driven by Dentour's Claude-skill commands (`/opsx:apply`, `/opsx:archive`). The validate command is *defined* and intended to gate archives, but **today's `main` is not gated by it**. Phase 6 design should treat wiring validate into CI as a possible studio contribution rather than a constraint to satisfy.
5. **`openspec/AGENTS.md` does not exist.** The tasks.md "if exists" hedge resolves to "not present". Authoring rules live in `openspec/config.yaml § rules`. Read that instead.
6. **No active changes at HEAD.** `openspec/changes/` contains only an empty `archive/` directory. The HEAD commit (2026-04-30) is the bootstrap of the workflow itself; no proposal has been opened since. **The studio's first delta will be the first delta period on this corpus**, with no worked example in-tree to mirror.
7. **Commit-scope enum will hard-fail on `(onboard-dentour)`.** Dentour's `commitlint.config.js` is severity-2 on `scope-enum`. Vivod is severity-1 (warn-but-pass). Phase 6 convention adapter design must either (a) generate commits scoped via an allowed value (`docs`, `workspace`, `features`, or one of the cross-cutting feature scopes), or (b) include a pre-engagement PR to Dentour that registers a `forge` / `studio` scope. The cheapest path is (a); record this in the convention-adapter design.
8. **Stripe is env-scaffolded, not implemented.** Despite proposal § Why listing Stripe, there is no `stripe` npm dep, no `apps/api/src/**/*stripe*` source, and no payments flow in the request path. Payments are a planned future capability; the convention adapter should not pre-emptively author payment-related specs until kickoff confirms the timeline.
9. **Redis is an undocumented integration.** `apps/api/package.json` declares `ioredis ^5.9` and `cache-manager-ioredis-yet ^2.1`; no governance doc (CLAUDE.md / proposal / design) mentions Redis. The drift audit should locate the Redis use sites and infer whether it is an opaque cache or part of the request-path contract.
10. **Turborepo is still v1.x.** `turbo ^1.11.0` with the v1 `pipeline.*` config shape. Vivod uses Turborepo 2 with `tasks.*`. The convention adapter design and any "studio runs Dentour's deploy" automation must use v1 syntax until Dentour upgrades.
11. **`auth-portal` is vestigial.** Only a `tsconfig.tsbuildinfo` file remains. Phase 6 adapter must blacklist this folder from delta generation. Infisical also retains a `/auth-portal` path which should not be referenced.
12. **`@dentour/api-client` is generated but unused.** Each frontend ships its own fetch wrapper. Any spec change that touches "the API client" must be explicit about which surface — generated package, or per-app wrapper. CLAUDE.md § 9 A1 is the canonical reference.

---

## 10. Source-of-truth pointers

Anyone reading this doc can re-derive everything from these files at the anchored commit (`2e96f677b7557b59530fbae87387b28e088defe3`):

- Repo root: [`README.md`](https://github.com/dentour-org/dentour/blob/2e96f67/README.md), [`CLAUDE.md`](https://github.com/dentour-org/dentour/blob/2e96f67/CLAUDE.md), [`BRANCHING_STRATEGY.md`](https://github.com/dentour-org/dentour/blob/2e96f67/BRANCHING_STRATEGY.md), [`package.json`](https://github.com/dentour-org/dentour/blob/2e96f67/package.json), [`pnpm-workspace.yaml`](https://github.com/dentour-org/dentour/blob/2e96f67/pnpm-workspace.yaml), [`turbo.json`](https://github.com/dentour-org/dentour/blob/2e96f67/turbo.json), [`commitlint.config.js`](https://github.com/dentour-org/dentour/blob/2e96f67/commitlint.config.js).
- Agent guidance: [`.github/copilot-instructions.md`](https://github.com/dentour-org/dentour/blob/2e96f67/.github/copilot-instructions.md). (No `openspec/AGENTS.md` at HEAD.)
- OpenSpec corpus: [`openspec/INDEX.md`](https://github.com/dentour-org/dentour/blob/2e96f67/openspec/INDEX.md), [`openspec/config.yaml`](https://github.com/dentour-org/dentour/blob/2e96f67/openspec/config.yaml), [`openspec/specs/`](https://github.com/dentour-org/dentour/tree/2e96f67/openspec/specs).
- CI: [`.github/workflows/`](https://github.com/dentour-org/dentour/tree/2e96f67/.github/workflows).
- Schema (canonical models): [`packages/database/prisma/schema.prisma`](https://github.com/dentour-org/dentour/blob/2e96f67/packages/database/prisma/schema.prisma).
- Domain registry (7 domains): [`packages/routes/src/constants/domains.ts`](https://github.com/dentour-org/dentour/blob/2e96f67/packages/routes/src/constants/domains.ts).
- Multi-currency formatter: [`packages/types/src/services.ts`](https://github.com/dentour-org/dentour/blob/2e96f67/packages/types/src/services.ts).

This document is authored by the studio for the studio. No reciprocal artifact is expected from Dentour until kickoff.
