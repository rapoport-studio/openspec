# Vivod Topology — Pre-engagement Scout

> Studio-internal topology scan of `VIVOD-AI/vivod`, anchored to a known commit. Captured during Phase 1 of [`onboard-vivod`](../../changes/onboard-vivod/). Read-only — nothing in this document was committed back to Vivod's repo.

```yaml
scout_date: 2026-05-12
vivod_commit_sha: 3f678eb88a7fc3ae97bfeae082f0b807e96a4734
vivod_commit_subject: 'chore(web): remove dead project mutation hooks (2026-04-10)'
vivod_repo_url: https://github.com/VIVOD-AI/vivod
governance_files_read:
  - CLAUDE.md
  - AGENTS.md
  - .github/copilot-instructions.md (header only)
  - openspec/INDEX.md
  - openspec/specs/index.md
  - openspec/specs/meta.json
  - package.json (root + per app/package)
  - pnpm-workspace.yaml
  - turbo.json
  - commitlint.config.cjs
  - Dockerfile (root)
  - railway.toml
  - .github/workflows/* (12 files listed)
  - docs/release-workflow.md
shareability: studio-internal-default
author: forge-subagent
companion_docs:
  - vivod-drift-audit.md (Phase 2 — not yet authored)
  - vivod-convention-adapter-design.md (Phase 6 — not yet authored)
```

---

## 1. Repo identity and source of truth

- **Canonical repo:** `VIVOD-AI/vivod` (confirmed via [`AGENTS.md`](https://github.com/VIVOD-AI/vivod) header line `**GitHub:** VIVOD-AI/vivod`).
- **Production URL:** `vivod.ai`. Subdomain split: `app.vivod.ai` (manager dashboard), `worker.vivod.ai` (mobile worker app), `portal.vivod.ai` (client/observer), `admin.vivod.ai` (back-office), `docs.vivod.ai` (Fumadocs portal), `api.vivod.ai` (HTTP gateway worker), `scope|guard|ledger|clerk|forge.vivod.ai` (agent workers), `cdn.vivod.ai` (R2 origin).
- **Branching model:** `feature/* → staging → main` (PR-only; no direct pushes). `main` auto-deploys; `staging` is a CI gate with no deploy. Hotfixes branch from `main`. Source: [`docs/release-workflow.md`](https://github.com/VIVOD-AI/vivod/blob/main/docs/release-workflow.md).
- **Commit convention:** Conventional Commits enforced by `commitlint.config.cjs` + Husky pre-commit hook. Type enum: `feat | fix | refactor | chore | perf | docs | test | ci | build | style`. Scope enum (severity 1, warn-only): `web | ui | db | domain | auth | agents | config | i18n | facade-engine | cloudflare | api-gateway | scope | ledger | clerk | ci | deps`. Subject MUST be lowercase (severity 2). Header max 100 chars (severity 2). **Scope enum omits `guard` and `clerk` despite both being live workers** — flag for the convention adapter; commits scoped `(guard)` or `(clerk)` will warn-but-pass today.
- **Package manager:** `pnpm@9.15.0` (pinned via `packageManager` field). Node engine `>=22`.
- **Monorepo orchestrator:** Turborepo 2 (`turbo.json` defines `build / dev / lint / check-types / test / test:e2e / gen-types / clean / storybook:dev / storybook:build`).
- **OpenSpec tooling:** `@fission-ai/openspec ^1.2.0` is a dev-dep at the repo root. `pnpm openspec validate` is run via the wrapper `scripts/openspec-validate.sh` (see § 7).

---

## 2. Deployed apps inventory

Vivod ships **three deployment classes**: one Railway-hosted Next.js app, six Cloudflare Workers (five wired to CI; one unwired), two Railway-hosted Python/Node renderer microservices, plus two non-deployed support apps (`storybook`, `e2e`). The `apps/web/wrangler.toml` indicates a Cloudflare-Workers-via-OpenNext deploy path also exists for `vivod-app`, but the production deploy goes via Railway per [`Dockerfile`](https://github.com/VIVOD-AI/vivod/blob/main/Dockerfile) + [`railway.toml`](https://github.com/VIVOD-AI/vivod/blob/main/railway.toml). This is a **dual-target build** that the topology should remember when sizing the convention adapter.

### 2.1 Next.js app — `apps/web` (`@vivod/web`)

- **Framework:** Next.js 15 with React 19, Tailwind CSS 4, TypeScript 5.5+ (strict).
- **Routing:** App Router with subdomain → route-group mapping in `src/middleware.ts`. Route groups under `src/app/[locale]/`:
  - `(platform)` — `app.vivod.ai` (manager/foreman dashboards, projects, contracts, payments)
  - `(worker)` — `worker.vivod.ai` (mobile-first today/reports/earnings)
  - `(worker-welcome)` — onboarding for new workers (no bottom-tab shell)
  - `(portal)` — `portal.vivod.ai` (client/observer read-only views)
  - `(admin)` — `admin.vivod.ai` (rewritten internally to `/admin/*` per REQ-AUTH-013)
  - `(marketing)` and `(landing)` — vivod.ai homepage / marketing
  - `(docs)` — Fumadocs MDX-driven internal spec portal
- **Locale routing:** `[locale]` segment (`he | ru | en`); Hebrew is the source of truth, RTL layout for `he`.
- **Standalone server:** Built via `next build` and packaged in a multi-stage Dockerfile (`turbo prune @vivod/web --docker` → `pnpm install --frozen-lockfile` → `pnpm build` → `node apps/web/server.js`).
- **Cloudflare path (secondary, currently unused for prod):** `pnpm preview` and `pnpm deploy` go through `@opennextjs/cloudflare` 1.17.1 with `apps/web/wrangler.toml` declaring worker name `vivod-app` and route `app.vivod.ai/*`. Useful evidence that the team intended (and may still intend) to migrate off Railway, but as of HEAD, **production traffic is served from Railway**, not Workers.
- **Observability:** Sentry (`@sentry/nextjs ^10.45`), PostHog (`posthog-js ^1.364`, `posthog-node ^5.28`).
- **Data fetching:** TanStack Query 5 (`@tanstack/react-query ^5.95`); query keys factory standard documented in `openspec/specs/api/query-layer.md`.
- **Auth:** `@supabase/ssr ^0.6` + `@vivod/auth` wrapper (phone OTP, PIN, magic link).
- **i18n:** `next-intl ^4.8` + custom `@vivod/i18n` package.

### 2.2 Cloudflare Workers — `apps/workers/*`

| Directory | Package name | Subdomain | CI deploy? | Purpose |
|---|---|---|---|---|
| `api-gateway` | `@vivod/worker-api-gateway` | `api.vivod.ai` | yes | Public Bearer-token gateway for browser & worker app; rate-limits + caches + forwards. KV: `SESSIONS`, `CACHE`, `RATE_LIMIT`. R2: `PHOTOS`, `DOCUMENTS`. |
| `scope` | `@vivod/worker-scope` | `scope.vivod.ai` | yes | SCOPE agent — facade photo analysis (Claude Sonnet 4 + image inputs). KV: `CONFIG`, `CACHE`. R2: `PHOTOS`. Var: `OPENCV_SERVICE_URL`. |
| `guard` | `@vivod/worker-guard` | (intended `guard.vivod.ai`) | **no** | GUARD agent — risk scoring, damage progression. Folder + `wrangler.toml` exist; **excluded from `deploy-workers.yml` matrix** (see § 5.2). Treat as draft. |
| `ledger` | `@vivod/worker-ledger` | `ledger.vivod.ai` | yes | LEDGER agent — finance, monthly count automation. Read-only against most tables. |
| `clerk` | `@vivod/worker-clerk` | `clerk.vivod.ai` | yes | CLERK agent — proposal/contract/document generation. KV: `CACHE`. R2: `DOCUMENTS`. Var: `TYPST_RENDERER_URL`. |
| `forge` | `@vivod/worker-forge` | `forge.vivod.ai` | yes | FORGE agent worker — bridges the `@vivod/forge-cli` package to Cloudflare runtime; orchestrates plan/audit/review pipelines. |
| `marketing` | `@vivod/worker-marketing` | (intended marketing static) | **no** | Static marketing site shim (Hono). Folder exists; **excluded from CI matrix**. |

All workers use the **Hono 4.7** framework on the `2025-03-01` compatibility date with `nodejs_compat`. Auth flow on protected endpoints: `Authorization: Bearer <jwt>` → `supabase.auth.getUser(token)` → look up `workers` row by `auth_user_id` → attach `AuthContext` to Hono context. Health endpoints (`/health`, `/v1/health`) are auth-exempt.

### 2.3 Railway-hosted renderer microservices — `services/*`

These are **outside the pnpm workspace**, deployed separately to Railway. They are referenced from the agent workers via `*_SERVICE_URL` vars (e.g., `OPENCV_SERVICE_URL` for SCOPE, `TYPST_RENDERER_URL` for CLERK).

| Directory | Stack | Purpose |
|---|---|---|
| `services/typst-renderer` | Node + Typst (Dockerfile + `server.mjs` + `package.json`) | Hebrew RTL PDF generation for contracts/proposals/reports. |
| `services/weasyprint-renderer` | Python (Dockerfile + `server.py` + `requirements.txt`) | Alternative PDF rendering path (HTML → PDF via WeasyPrint). |

There is also an `apps/services/` directory containing **`opencv` and `reconstruction`** subfolders — separate from `services/`. Their relationship to the build is not yet inspected; **flag for Phase 2 drift audit**.

### 2.4 Non-deployed support apps

| Directory | Package name | Purpose |
|---|---|---|
| `apps/storybook` | `@vivod/storybook` | Storybook 10.3 with Vite 4 + Tailwind 4 + Chromatic; consumes `@vivod/ui`, `@vivod/config`, `@vivod/domain`, `@vivod/facade-engine`, `@vivod/i18n`. Ships visual-regression baselines via Chromatic (`pnpm chromatic`). |
| `apps/e2e` | `@vivod/e2e` | Playwright 1.50 test suite; tags `@smoke`, `@auth`, `@platform`. Has its own Sentry client for synthetic-monitoring runs. Triggered by `e2e-monitor.yml` workflow. |

---

## 3. Packages inventory

Ten internal packages under `packages/` plus a small auxiliary package (`@vivod/cloudflare`) shared by all workers. All packages are `private: true` with `version: "0.0.x"`. Build tooling: `tsup ^8` for libraries that emit `dist/`; some packages (`auth`, `cloudflare`, `i18n`) export directly from `src/` and skip the build step.

| Package | Build output | Subpath exports | Public API summary |
|---|---|---|---|
| `@vivod/domain` | `dist/index.js` (tsup) | `.` only | Branded ID types (`OrganizationId`, `ProjectId`, …, `MaterialReturnId`); enums (`BlockStatus`, `WorkerRole`, …); Zod schemas; `Money = { agorot: number, currency: 'ILS' }`; `routes` registry; `AppError`; agent metadata (`AGENT_META`). **Zero internal deps** (only `libphonenumber-js`, `zod`). |
| `@vivod/config` | `dist/` (tsup) | `./eslint`, `./tailwind`, `./tokens`, `./tokens/colors.css`, `./tokens/motion.css`, `./env`, `./tsconfig/{base,next,library}` | Tailwind 4 token set (CSS variables in 6 layers — see `design/index.md` § REQ-DSN-002), ESLint flat config, env validators (Zod), tsconfig presets. **Zero internal deps.** |
| `@vivod/db` | `dist/index.js` (tsup) | `.` only | Supabase client factories (`createServerClient`, browser client), per-table query functions (`packages/db/src/queries/*`), `snakeToCamelRows` mapper, generated `types.ts`. Imports `@vivod/domain`, `@vivod/config`. |
| `@vivod/auth` | `src/` (no build) | `.`, `./middleware`, `./components/*`, `./hooks/*`, `./providers/*`, `./client`, `./actions` | `withAuth`, `withRole`, `withAdminOnly` wrappers; `AuthContext`; PIN/OTP flows; `RoleGate`, `AuthProvider`; HMAC header signing (`AUTH_HEADER_HMAC_SECRET`). Imports `@vivod/db`, `@vivod/domain`, `@vivod/i18n`, `@vivod/ui`, `@vivod/config`. Peer-deps `next ^15`, `react ^19`. |
| `@vivod/i18n` | `src/` (no build) | `.`, `./react`, `./messages/{he,ru,en}` | 22 namespaces, ~379 keys × 3 locales (`he` source of truth). `useTranslations()` (client) + `getMessagesSync()` (server). RTL handling baked in. **No internal deps.** |
| `@vivod/ui` | `dist/index.js` (tsup) + `./globals.css` | `.` (barrel) + globals | shadcn/ui primitives in `primitives/` + composed VIVOD components in `composed/`. Heavy Radix surface (`@radix-ui/react-{accordion,collapsible,dialog,dropdown-menu,navigation-menu,select}`), `cva`, `lucide-react`, `recharts`, `react-hook-form`, `react-phone-number-input`. Imports `@vivod/config`, `@vivod/domain`, `@vivod/facade-engine`, `@vivod/i18n`. |
| `@vivod/facade-engine` | `dist/` (tsup) | `.` only | SVG-based facade grid renderer (`BlockCellV2`); shares `BlockVisualData` with `BlockCellHtml` in `@vivod/ui` for dual-renderer parity. Imports `@vivod/domain`. |
| `@vivod/agents` | `dist/` (tsup) | `.` only | Anthropic SDK orchestration (`@anthropic-ai/sdk ^0.39`) shared between web (server actions) and workers. Imports `@vivod/domain`, `@vivod/db`. |
| `@vivod/forge-cli` | `dist/` + `bin: forge` | `.`, `./core`, `./worker-prompts` | The Vivod **internal** Forge CLI (their executor — not to be confused with Rapoport Studio's Forge). `chalk ^5`, `glob ^11`, custom plan/audit/review commands. Imports `@vivod/db`, `@vivod/domain`. Surfaced to web via `apps/web/package.json` dependency `@vivod/forge-cli` and to the `forge` worker. Persists results via `scripts/forge-persist.mjs`. |
| `@vivod/cloudflare` | `src/` (no build) | `.`, `./auth`, `./cors`, `./cache`, `./r2`, `./rate-limit`, `./supabase` | Hono helpers shared by every worker (auth middleware, CORS, KV cache, R2, rate limiting, Supabase service-role client). **No internal deps** — pure infra utility. |

### 3.1 Internal dependency graph

The graph is shallow and well-layered (no cycles observed across the listed `package.json` deps):

```
                       ┌────────────────────┐
                       │  @vivod/domain     │  (zero internal deps)
                       └──┬──────────────┬──┘
                          │              │
            ┌─────────────┘        ┌─────┴─────────┐
            ▼                      ▼               ▼
    ┌──────────────┐        ┌────────────┐   ┌──────────────┐
    │ @vivod/db    │        │ @vivod/    │   │ @vivod/      │
    │ (uses domain,│        │ facade-    │   │ agents       │
    │  config)     │        │ engine     │   │ (anthropic)  │
    └──────┬───────┘        └─────┬──────┘   └──────┬───────┘
           │                      │                 │
           ▼                      ▼                 │
    ┌──────────────┐        ┌──────────────┐        │
    │ @vivod/auth  │        │ @vivod/ui    │        │
    │ (db, domain, │◀───────┤ (config,     │        │
    │  i18n, ui,   │        │  domain,     │        │
    │  config)     │        │  facade-     │        │
    └──────┬───────┘        │  engine,     │        │
           │                │  i18n)       │        │
           │                └──────┬───────┘        │
           │                       │                │
           │   ┌───────────────────┼────────────────┘
           │   │                   │
           ▼   ▼                   ▼
        ┌──────────────────────────────┐
        │  @vivod/web                  │  + also imports @vivod/forge-cli
        │  (auth, db, domain, ui, i18n,│
        │   facade-engine, agents,     │
        │   forge-cli)                 │
        └──────────────────────────────┘

  ┌─────────────────────┐         ┌──────────────────┐
  │ @vivod/cloudflare   │◀────────┤ apps/workers/*   │  (each also pulls
  │ (hono, supabase-js) │         │ (api-gateway,    │   @supabase/supabase-js
  └─────────────────────┘         │  scope, guard,   │   except marketing;
                                  │  ledger, clerk,  │   forge worker uniquely
                                  │  forge,          │   pulls @vivod/forge-cli)
                                  │  marketing)      │
                                  └──────────────────┘

  ┌─────────────────────┐
  │ @vivod/i18n         │  consumed by auth, ui, web, storybook
  │ (zero internal)     │
  └─────────────────────┘

  ┌─────────────────────┐
  │ @vivod/forge-cli    │  consumed by web (server actions / API routes)
  │ (db, domain)        │  and by apps/workers/forge (Hono runtime)
  └─────────────────────┘

  ┌─────────────────────┐
  │ apps/storybook      │  consumes config, domain, facade-engine, i18n, ui
  └─────────────────────┘

  ┌─────────────────────┐
  │ apps/e2e            │  no workspace deps; uses @supabase/supabase-js +
  │                     │  @sentry/node + @playwright/test directly
  └─────────────────────┘
```

Key invariants observed:
- **No `apps/web → apps/workers/*` import path exists** (workers are deployed independently and called via HTTP only).
- **No worker imports `apps/web`.** Workers import `@vivod/cloudflare`, `@supabase/supabase-js`, `hono` — and the `forge` worker uniquely imports `@vivod/forge-cli` for command logic.
- **`@vivod/cloudflare` is the only package that is workers-only.** Every other shared package is consumed by `apps/web` (the Next.js side).
- **Branded ID types in `@vivod/domain` enforce inter-package safety** at compile time (e.g., a `ProjectId` cannot be passed where `BuildingId` is expected — see REQ-ENT-001).

---

## 4. Entry points and runtime topology

| Surface | Entry file | Runtime | Triggered by |
|---|---|---|---|
| Web SSR/RSC | `apps/web/src/app/[locale]/layout.tsx` (+ middleware `apps/web/src/middleware.ts`) | Node 22 (Railway container) | HTTP traffic via Railway; middleware detects subdomain and route group. |
| Web standalone server | `apps/web/server.js` (generated by `next build`) | Node 22 | `CMD` in root `Dockerfile`. |
| Web → Cloudflare (alt) | `apps/web/.open-next/worker.js` | Workers + `nodejs_compat` | `pnpm preview` / `pnpm deploy` (not currently in CI). |
| Worker `api-gateway` | `apps/workers/api-gateway/src/index.ts` | Workers (smart placement) | Bearer-token requests on `api.vivod.ai/v1/*`. |
| Worker `scope` | `apps/workers/scope/src/index.ts` | Workers | `POST scope.vivod.ai/analyze/{photo,corners}`, `POST /rectify/facade`. |
| Worker `guard` | `apps/workers/guard/src/index.ts` | Workers (folder exists, **not in deploy matrix**) | TBD — risk scoring; orchestrated via `processScopeDamages` server action per REQ-AGT-013. |
| Worker `ledger` | `apps/workers/ledger/src/index.ts` | Workers | `POST ledger.vivod.ai/finance/*`, `/velocity/*`, `/payroll/*`. |
| Worker `clerk` | `apps/workers/clerk/src/index.ts` | Workers | `POST clerk.vivod.ai/generate/proposal`, `/documents/*`. |
| Worker `forge` | `apps/workers/forge/src/index.ts` | Workers | Internal CLI bridge for the FORGE agent pipeline. |
| Forge CLI | `packages/forge-cli/dist/cli.js` (`bin: forge`) | Node 22 | `pnpm forge <subcommand>` and `pnpm forge:audit`. |
| Storybook | `apps/storybook` (Vite) | Static | `pnpm storybook` (dev), `pnpm storybook:build` (output `dist/`). |
| Playwright E2E | `apps/e2e/playwright.config.ts` | Node + Chromium | `pnpm test:e2e` and the `e2e-monitor.yml` GitHub workflow. |
| Typst renderer | `services/typst-renderer/server.mjs` | Node (Railway, separate service) | Called via `TYPST_RENDERER_URL` from CLERK. |
| WeasyPrint renderer | `services/weasyprint-renderer/server.py` | Python (Railway, separate service) | Alternative PDF path. |

---

## 5. Deploy targets

Vivod's deploy surface is **multi-target and asymmetric**: one Railway container for `apps/web`, five Cloudflare Workers in CI, two unwired workers, two Railway micro-services. There is no single `deploy` command — different paths use different actions.

### 5.1 Web app — Railway

- **Trigger:** push to `main` with `paths-ignore: ['apps/workers/**']` (workflow `deploy.yml`).
- **Build env:** `infisical run --env=prod --path=/ --recursive --projectId $INFISICAL_PROJECT_ID --domain https://eu.infisical.com -- pnpm build`. Build-time vars validated against required list (`NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `NEXT_PUBLIC_SITE_URL`, `NEXT_PUBLIC_APP_URL`).
- **Deploy step:** `npm i -g @railway/cli@latest && railway up --service vivod` with 3-attempt retry (20s/40s/60s backoff).
- **Production runtime:** standalone Next.js server inside `node:22-alpine`, ENV `NODE_ENV=production`, `EXPOSE 3000`. Runs as non-root `nextjs:nodejs` (UID 1001).
- **Healthcheck:** Railway calls `/api/health` (per `railway.toml`).

### 5.2 Cloudflare Workers — wrangler-action

- **Trigger:** push to `main` matching `apps/workers/**`, `packages/cloudflare/**`, `packages/domain/**`, `packages/db/**`, `pnpm-lock.yaml` (workflow `deploy-workers.yml`); also `workflow_dispatch` with worker picker.
- **Detection:** `dorny/paths-filter@v3` produces a per-worker boolean; if any of `cloudflare/db/domain/lockfile` changed, all workers rebuild (the "shared" trip-wire).
- **Matrix:** `[api-gateway, scope, ledger, clerk, forge]` — **5 workers**. **`guard` and `marketing` are intentionally absent from the matrix** even though their folders, `wrangler.toml`, and worker `package.json` exist. Studio assumption: `guard` is in flight as a draft and `marketing` is replaced by the `(marketing)` route group inside `apps/web` — but this is **inferred and worth confirming with Vivod at kickoff**.
- **Steps per worker:** `pnpm install --frozen-lockfile` → `pnpm --filter @vivod/forge-cli build` → `npx tsc --noEmit` (per `apps/workers/<w>`) → `cloudflare/wrangler-action@v3 deploy` → 3-attempt health probe against the worker's domain.
- **Secrets:** `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID`. Per-worker secrets (`SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `SUPABASE_JWT_SECRET`, `ANTHROPIC_API_KEY`) are set via `wrangler secret put` out-of-band, sourced from Infisical.

### 5.3 Renderer microservices — Railway

Both `services/typst-renderer` and `services/weasyprint-renderer` ship as their own Dockerfiles to Railway. They are **outside the pnpm workspace** and therefore outside the Turbo cache. No GitHub workflow inspected so far covers them — likely Railway-watched repos or manual deploy. **Flag for Phase 2 verification.**

### 5.4 Supabase — `pnpm db:push`

- DB is Supabase Postgres with RLS on **all 14 tables**, scoped by `auth_org_id()` (REQ-INF-010, REQ-AUTH-039).
- Migrations under `supabase/migrations/`; six migrations counted at HEAD.
- Apply via `npx supabase db push` (root `package.json` script `db:push`); types regenerated via `db:gen-types`. Per [`AGENTS.md`](https://github.com/VIVOD-AI/vivod/blob/main/AGENTS.md): "always use this, not MCP `apply_migration` — it reloads PostgREST".
- A `complete_onboarding` SECURITY DEFINER RPC handles the org/worker chicken-and-egg (REQ-AUTH-023 / REQ-AUTH-040).

---

## 6. CI / CD inventory

`.github/workflows/` contains 12 files:

| Workflow | Trigger | Purpose |
|---|---|---|
| `ci.yml` | PR to `main` or `staging` | Lint + check-types + test + build (with Infisical staging secrets). The merge gate. |
| `deploy.yml` | Push to `main` (excluding workers) | Build with prod Infisical secrets, deploy `apps/web` to Railway. |
| `deploy-workers.yml` | Push to `main` matching workers/cloudflare/domain/db/lockfile | Per-worker matrix deploy via wrangler-action with health probes. |
| `pr-workers.yml` | PR touching workers | Type-check workers without deploying (preview gate). |
| `preview.yml` | PR | Preview build (likely Cloudflare preview URL — not inspected line-by-line). |
| `release.yml` | Push to `main` | Release-Please-driven release notes (`release-please-config.json` exists at root). |
| `e2e-monitor.yml` | Schedule + on-demand | Synthetic Playwright runs against production. |
| `forge-execute.yml` | `workflow_dispatch` | Runs Vivod's internal Forge CLI in CI to execute an approved plan against a Linear issue (`VVD-*`). Permissions: `contents: write`, `pull-requests: write`. Uses `FORGE_GITHUB_PAT`. |
| `forge-pr-review.yml` | PR `opened/synchronize` to `main` | Runs Forge `review` on the PR diff and posts findings as a comment. **Informational, not a required check.** Uses `ANTHROPIC_API_KEY`. |
| `forge-rules.yml` | (not inspected line-by-line) | Likely runs the rule scanner under `.github/forge/rules/`. |
| `forge-weekly-audit.yml` | Schedule | Weekly Forge audit pipeline (cost report / spec drift). |

Notable absences and gotchas for the convention adapter:
- **No workflow runs `pnpm openspec validate`.** Validation is run locally via `scripts/openspec-validate.sh` which **temporarily symlinks `index.md → spec.md`** for each spec dir that contains `## Purpose` or `## Requirements`, then invokes `npx openspec validate --all --strict`, then cleans up. This is a **filename-bridge dance** between Fumadocs (which wants `index.md`) and the OpenSpec CLI (which wants `spec.md`). The convention adapter must understand this — **emitting only `index.md` and relying on the bridge script is the established Vivod pattern, not a bug**.
- **`forge-pr-review.yml` is informational only.** The studio's Hybrid-mode bot (per `onboard-vivod` design.md § Engagement mode default) will land its specs as PRs that pass through this commenter. Behavior to verify at kickoff: does the studio bot's account get a Forge review, and is that desirable?
- **Commit scope warning.** `commitlint.config.cjs` warns (severity 1) when scope ∉ enum; this means studio commits scoped `(onboard-vivod)` or `(forge-vivod-adapter)` will warn-but-pass. Acceptable under Hybrid mode if Vivod accepts the warning.

---

## 7. OpenSpec corpus

### 7.1 Layout

Vivod uses `openspec/specs/<capability>/` as the source-of-truth tree. **There is no `openspec/current/` folder**, contrary to the studio's convention. Every capability is a folder; most expose `index.md` (Fumadocs MDX with frontmatter), and a subset additionally expose `spec.md` for the OpenSpec CLI to consume directly. Where `spec.md` is missing, `scripts/openspec-validate.sh` materializes it temporarily from `index.md` (see § 6 gotcha).

```
openspec/
├── INDEX.md                         # Human-facing inventory (4 standalone + 12 module specs)
├── specs/                           # Source of truth
│   ├── index.md                     # Fumadocs landing
│   ├── meta.json                    # Fumadocs page order
│   ├── design/index.md              # 44 REQ-DSN-* requirements
│   ├── auth/index.md                # 47 REQ-AUTH-* requirements
│   ├── api/
│   │   ├── index.md                 # 37 REQ-API-* requirements
│   │   └── query-layer.md           # 14 REQ-QL-* requirements (TanStack Query standard)
│   ├── platform/
│   │   ├── index.md                 # Landing only (links to subsections)
│   │   ├── analytics.md             # Standalone: 9 REQ-PH-* (PostHog)
│   │   ├── entities/                # project-card, project-row sub-pages
│   │   └── pages/                   # project-detail, projects-list sub-pages
│   ├── facade/spec.md               # FacadeTrack — 65 REQ-FCD-* requirements
│   ├── agents/
│   │   ├── spec.md                  # 82 "Requirement REQ-AGT-*" rows (note label variant)
│   │   └── ai-architecture.md       # 28 cross-cutting requirements (orchestration, model selection, costs)
│   ├── infra/
│   │   ├── index.md                 # 14 REQ-INF-* requirements
│   │   └── {agents-infra,deployment,domains,monitoring,secrets}/
│   ├── contracts/spec.md            # 47 REQ-CON-* requirements (matches design.md claim)
│   ├── worker-app/spec.md           # 40 REQ-WKR-* requirements
│   ├── forge-console-ui/index.md    # 20 REQ-P3-* requirements (FORGE console UI Phase 3)
│   ├── entities/spec.md             # 73 REQ-ENT-* requirements (13 core entities)
│   ├── ui/                          # 18 sub-capabilities — each with index.md
│   │   ├── primitives/, forms/, navigation/, data-display/, misc/
│   │   ├── entity-views/, entity-avatar/, entity-card/, entity-row/
│   │   ├── collection-empty/, collection-engine/, collection-toolbar/
│   │   ├── facade/, contract/, task/, worker/
│   │   ├── segmented-bar/, stat-cell/, status-badge/, wizard/
│   │   └── meta.json
│   ├── forms/project-form.md        # Standalone: 8 REQ-PF-* requirements
│   └── pages/
│       ├── create-project-page.md   # Standalone — Fumadocs frontmatter
│       └── edit-project-page.md     # Standalone — Fumadocs frontmatter
└── changes/                         # Active proposals (formal-deltas style, opposite of studio)
    ├── add-product-charter/
    ├── add-realtime-sync-and-route-cache/
    ├── archive/                     # Historical
    ├── draft-product-character/
    ├── forge-console-ui/
    ├── simplify-project-creation/
    └── worker-today-view/
```

### 7.2 Module spec inventory (matches `INDEX.md` + onboard-vivod proposal claim)

| # | Module | File | Requirement count | Notes |
|---|---|---|---|---|
| 1 | Design | `design/index.md` | 44 (`REQ-DSN-*`) | 6-layer color system, 83 CSS vars, RTL typography. |
| 2 | Auth | `auth/index.md` | 47 (`REQ-AUTH-*`) | Phone OTP / PIN / magic link / RBAC (7 roles). |
| 3 | API | `api/index.md` (+ `query-layer.md` annex) | 37 (`REQ-API-*`) + 14 (`REQ-QL-*`) | `withAuth/withRole/withAdminOnly` wrappers. |
| 4 | Platform | `platform/index.md` (+ subsection pages) | (variable — pages, entities) | Aggregates entity views + page specs. |
| 5 | FacadeTrack | `facade/spec.md` | 65 (`REQ-FCD-*`) | Grid renderer, block status, dual-renderer. |
| 6 | Agents | `agents/spec.md` (+ `ai-architecture.md`) | 82 + 28 | Uses `### Requirement REQ-AGT-*` heading style (different from other specs). |
| 7 | Infra | `infra/index.md` (+ 5 sub-folders) | 14 (`REQ-INF-*`) | DNS, KV, R2, RLS, Infisical, CI/CD. |
| 8 | Contracts | `contracts/spec.md` | 47 (`REQ-CON-*`) | 9-state FSM, FMEA risk scoring, RTL PDF. |
| 9 | Worker App | `worker-app/spec.md` | 40 (`REQ-WKR-*`) | Mobile bottom-tab shell, Today view. |
| 10 | Forge Console UI | `forge-console-ui/index.md` | 20 (`REQ-P3-*`) | Vivod's own internal Forge dashboard UI. |
| 11 | UI Components | `ui/<sub>/index.md` × 18 | (sub-capability counts) | Primitives + composed VIVOD components. |
| 12 | Entities | `entities/spec.md` | 73 (`REQ-ENT-*`) | 13 core entities, 4-level view system, full ID prefix registry. |

### 7.3 Standalone specs (matches `INDEX.md`)

| # | Spec | File | Requirement count |
|---|---|---|---|
| 1 | Project Form Components | `forms/project-form.md` | 8 (`REQ-PF-*`) |
| 2 | Create Project Page | `pages/create-project-page.md` | (Fumadocs spec; not REQ-counted via grep) |
| 3 | Edit Project Page | `pages/edit-project-page.md` | (Fumadocs spec; not REQ-counted via grep) |
| 4 | Analytics Standard (PostHog) | `platform/analytics.md` | 9 (`REQ-PH-*`) |

### 7.4 Convention specifics for Forge adapter (Phase 6 input)

- **Heading style is not uniform.** Most specs use `### REQ-<DOMAIN>-NNN` (auth, api, infra, facade, contracts, worker-app, design, query-layer, analytics, project-form, forge-console-ui). The agents spec uses `### Requirement REQ-AGT-NNN` and the entities spec uses `### REQ-ENT-NNN`. The convention adapter parser must accept both prefixes (`### REQ-` and `### Requirement REQ-`).
- **Scenario blocks** are consistently formatted: `#### Scenario: <name>` followed by `- **WHEN** … - **THEN** …` bullets (sometimes with `- **GIVEN** …` preamble).
- **Acceptance Criteria tables** are at the end of most specs (`| REQ ID | Criterion | Pass condition |`). Auth has 47 rows, infra 24+, contracts has its own format. The adapter should preserve them but not require their presence (some specs omit).
- **`## Purpose`** at the top of every capability spec is mandatory by the validate-bridge script's heuristic.
- **Implementation map sections are inconsistent.** Some specs have `## Cross-References` linking to sibling specs; some have prose pointing into `apps/web/src/...` paths inline. There is **no canonical `## Implementation map` section name** as the design.md draft assumed — Phase 6 should standardize on `## Cross-References` or equivalent observed name, not invent a new one.
- **Fumadocs frontmatter** (`---\ntitle: ...\ndescription: ...\n---`) is required for the spec to render in `docs.vivod.ai`. The adapter must preserve frontmatter when editing.
- **Deltas live at `openspec/changes/<slug>/`** with formal sub-folders matching the source-of-truth shape (`changes/<slug>/specs/<cap>/spec.md`). This is the **opposite** of the studio's prose-style flat `proposal.md / design.md / tasks.md`. Confirmed by directory inspection of `openspec/changes/forge-console-ui/`, `worker-today-view/`, etc.

### 7.5 Active changes at HEAD

Six active changes plus an `archive/`:

```
add-product-charter, add-realtime-sync-and-route-cache, draft-product-character,
forge-console-ui, simplify-project-creation, worker-today-view
```

The `forge-console-ui` change is the source of the `REQ-P3-*` requirements that already landed in `specs/forge-console-ui/index.md` — confirms the **change → apply → archive** lifecycle is in active use.

---

## 8. External integrations

| Provider | Use | Wired via | Secret name(s) |
|---|---|---|---|
| **Supabase** | Postgres + Auth + Storage + Realtime; eu-central-2 region | `@supabase/supabase-js`, `@supabase/ssr` (web), service-role client in workers | `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, `SUPABASE_JWT_SECRET`, `SUPABASE_PROJECT_ID` |
| **Twilio** | SMS for phone OTP (REQ-AUTH-004 / REQ-AUTH-005) | Configured at the Supabase Auth layer (not a direct Vivod npm dep) | Provider config in Supabase dashboard |
| **Anthropic** | Claude Sonnet 4 for SCOPE/GUARD/CLERK/LEDGER/FORGE; image inputs for SCOPE | `@anthropic-ai/sdk ^0.39` (in `@vivod/agents` and worker bundles); FORGE Anthropic-CLI invocation in `forge-execute.yml` | `ANTHROPIC_API_KEY` |
| **Cloudflare** | Workers, KV (`SESSIONS / CACHE / RATE_LIMIT / CONFIG`), R2 (`vivod-photos / vivod-documents`), DNS for `vivod.ai`, TLS Full(strict) | `wrangler ^3` per worker, `@opennextjs/cloudflare ^1.17` for the alt web path | `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID`, per-worker `wrangler secret put` |
| **Railway** | Production hosting for `apps/web` + the two renderer services | `@railway/cli` invoked from `deploy.yml`; `Dockerfile` + `railway.toml` per service | `RAILWAY_TOKEN` |
| **Infisical** | Centralized secret vault (org "VIVOD Ltd"), env-scoped (`development / staging / prod`) | `infisical` CLI in CI workflows; `pnpm secrets:pull` for local dev (`infisical export --env=staging --path=/{supabase,anthropic,app}`) | `INFISICAL_CLIENT_ID`, `INFISICAL_CLIENT_SECRET`, `INFISICAL_PROJECT_ID` |
| **PostHog** | Product analytics + session replay (EU Cloud); custom events per `analytics.md` (`REQ-PH-*`); reverse proxy `ph.vivod.ai` planned but not wired | `posthog-js` (web client), `posthog-node` (server actions / API routes) | `NEXT_PUBLIC_POSTHOG_PROJECT_TOKEN`, `NEXT_PUBLIC_POSTHOG_HOST` (default `https://eu.i.posthog.com`) |
| **Sentry** | Error tracking + session replay; per-environment DSN for web; shared agents DSN for workers; dedicated DSN for the e2e synthetic monitor | `@sentry/nextjs` (web), `@sentry/node` (e2e) | `NEXT_PUBLIC_SENTRY_DSN`, `SENTRY_AUTH_TOKEN`, `SENTRY_ORG`, `SENTRY_PROJECT` |
| **Linear** | Issue tracking with `VVD-*` prefix; Forge plan execution targets a Linear issue | `forge-execute.yml` reads `inputs.issue_key` (e.g. `VVD-51`); persistence via `scripts/forge-persist.mjs` | (Linear API token referenced indirectly via Forge persist script — exact secret name not surfaced in workflow inspection; **flag for kickoff**) |
| **GitHub** | Code hosting (`VIVOD-AI/vivod`), Actions CI/CD, Release Please for changelogs | 12 workflows; Release-Please config at `release-please-config.json` | `FORGE_GITHUB_PAT` (for Forge to push branches & open PRs), `GITHUB_TOKEN` (default) |
| **Turborepo Remote Cache** | Build cache | `TURBO_TOKEN`, `TURBO_TEAM` env in every workflow | `TURBO_TOKEN`, `TURBO_TEAM` |
| **Chromatic** | Visual regression for Storybook | `apps/storybook` package script `chromatic` | (Chromatic project token — not surfaced in inspection) |
| **WhatsApp** | Worker invitation deep links (`https://wa.me/{phone}?text=...`) per REQ-AUTH-020; not a paid integration, just URL-builder | inline in `inviteWorker` action | none |
| **Typst** | RTL Hebrew PDF rendering | `services/typst-renderer` Railway service called via `TYPST_RENDERER_URL` | none (internal service URL) |
| **WeasyPrint** | Alternative HTML→PDF rendering | `services/weasyprint-renderer` Railway service | none (internal service URL) |
| **OpenCV** | Image preprocessing for SCOPE (corner detection, perspective correction) | `apps/services/opencv` referenced via `OPENCV_SERVICE_URL` (deploy mechanism for `apps/services/*` not yet inspected — **flag for Phase 2**) | none (internal service URL) |

Implicit: **`@cloudflare/workers-types`** (devDependency on every worker), **shadcn/ui + Radix** (UI primitives, not a runtime integration), **Husky + lint-staged + Commitlint** (commit-time hooks).

---

## 9. Studio-side observations and open flags

These are **studio-internal notes** for downstream phases of `onboard-vivod` — not assertions about Vivod, just things to verify or remember:

1. **Dual web target.** `apps/web` builds for both Railway (Dockerfile, in production) and Cloudflare (`apps/web/wrangler.toml` + `@opennextjs/cloudflare`, not in CI). The convention adapter and any future "studio runs Vivod's deploy" automation must know which is the active path.
2. **`guard` and `marketing` workers exist on disk but not in CI.** Treat as drafts. Don't generate deltas that assume `guard.vivod.ai` or `marketing.vivod.ai` are live without confirming.
3. **`commitlint.config.cjs` scope enum is missing `guard` and `clerk`.** Vivod commits scoped `(guard)` or `(clerk)` warn-but-pass today. If the studio bot lands `forge(onboard-vivod): …`-style commits via Hybrid mode, those will warn (the studio's `forge` scope is also not in the enum). Either register a studio-side scope or accept the warning at kickoff.
4. **No `openspec validate` runs in CI.** The `scripts/openspec-validate.sh` bridge is invoked manually (`pnpm forge:audit` or local pre-commit). The convention adapter design (Phase 6) should still call validate as a build gate, but recognize that **today's main is not gated by it** — drift between specs and the validator is possible.
5. **Heading-style heterogeneity.** `agents/spec.md` uses `### Requirement REQ-AGT-*`; everything else uses `### REQ-*`. Single-prefix parsers will miss the agents spec. Phase 6 parser must accept both.
6. **No `## Implementation map` section.** Cross-references live in `## Cross-References` tables. Don't invent a new section — reuse what's there.
7. **`apps/services/opencv` and `apps/services/reconstruction` are unexplored.** They sit outside the pnpm workspace folders the topology team usually walks. **Phase 2 drift audit** should catalog them before declaring the deploy-target list complete.
8. **Linear secret name for Forge unclear.** `forge-execute.yml` only surfaces `FORGE_GITHUB_PAT` and `ANTHROPIC_API_KEY`; Linear access is presumably handled inside `scripts/forge-persist.mjs` but the secret name is not visible in workflow YAML. **Resolve at kickoff** before the studio writes anything that calls Linear from inside the engagement repo.
9. **Renderer micro-service deploy mechanism not yet inspected.** `services/typst-renderer` and `services/weasyprint-renderer` likely have Railway service definitions outside the GitHub Actions inspected here. **Phase 2** should fill this in.
10. **Branch-shape compatibility check needed.** Vivod's working branch shape is `feature/VIV-123-slug` → `staging` → `main`. The studio's default `forge/<change-slug>/<task-slug>` shape will pass commitlint header checks but will not be a `feature/*` and will not have a `VIV-` prefix. Hybrid mode posts to a `studio/<slug>` branch that Vivod merges manually — this needs ratification at kickoff (recorded in `vivod.yaml` T4.1 row `engagement.handoff.code_branch_shape`).

---

## 10. Source-of-truth pointers

Anyone reading this doc can re-derive everything from these files at the anchored commit:

- Repo root: [`AGENTS.md`](https://github.com/VIVOD-AI/vivod/blob/3f678eb/AGENTS.md), [`CLAUDE.md`](https://github.com/VIVOD-AI/vivod/blob/3f678eb/CLAUDE.md), [`package.json`](https://github.com/VIVOD-AI/vivod/blob/3f678eb/package.json), [`pnpm-workspace.yaml`](https://github.com/VIVOD-AI/vivod/blob/3f678eb/pnpm-workspace.yaml), [`turbo.json`](https://github.com/VIVOD-AI/vivod/blob/3f678eb/turbo.json), [`commitlint.config.cjs`](https://github.com/VIVOD-AI/vivod/blob/3f678eb/commitlint.config.cjs), [`Dockerfile`](https://github.com/VIVOD-AI/vivod/blob/3f678eb/Dockerfile), [`railway.toml`](https://github.com/VIVOD-AI/vivod/blob/3f678eb/railway.toml).
- OpenSpec corpus: [`openspec/INDEX.md`](https://github.com/VIVOD-AI/vivod/blob/3f678eb/openspec/INDEX.md), [`openspec/specs/`](https://github.com/VIVOD-AI/vivod/tree/3f678eb/openspec/specs), [`scripts/openspec-validate.sh`](https://github.com/VIVOD-AI/vivod/blob/3f678eb/scripts/openspec-validate.sh).
- CI: [`.github/workflows/`](https://github.com/VIVOD-AI/vivod/tree/3f678eb/.github/workflows).
- Release process: [`docs/release-workflow.md`](https://github.com/VIVOD-AI/vivod/blob/3f678eb/docs/release-workflow.md).
- Wrangler configs: each worker's [`apps/workers/<w>/wrangler.toml`](https://github.com/VIVOD-AI/vivod/tree/3f678eb/apps/workers).

This document is authored by the studio for the studio. No reciprocal artifact is expected from Vivod until kickoff.
