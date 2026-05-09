# Rapoport Studio — Project Charter (handoff for new sub-products)

> **Read this first.** This document is the single hand-off artifact for an agent bootstrapping OpenSpec documentation in a **new sub-product of Rapoport Studio** (e.g. the psychology product). It describes the studio's operating model, the canonical stack, conventions, and the OpenSpec methodology used across every project the studio runs.
>
> Everything here is the truth as of 2026-05-05 and is derived from `openspec/current/platform.md` and `openspec/AGENTS.md` of the platform repo. If something here contradicts those files in the future, those files win.

---

## 0. How to use this document

You are an agent. You have just been given access to a **new repository** for a new product that lives under the Rapoport Studio umbrella (the immediate consumer is a psychology-domain product). The product does not yet have an OpenSpec corpus. Your task is to:

1. Read this document end-to-end.
2. Read whatever discovery / brief / Linear material the user gives you alongside it.
3. Generate the new project's `openspec/current/platform.md` (and capability stubs) **in the same prose style and shape** as the studio platform — not as formal `specs/` deltas.
4. Surface, in writing, every place where the new product needs a divergence from this charter (examples: different jurisdiction, different data sensitivity, different runtime). Do not silently invent.

When in doubt about shape or tone, **mirror `openspec/current/platform.md` of the platform repo** — it is the canonical example.

---

## 1. What Rapoport Studio is

Rapoport Studio is a Moldovan SRL (in formation, MITP applicant) that builds digital platforms and AI-powered infrastructure for businesses, corporations, and government entities.

The studio operates as a **curator of teams**: each engagement gets a hand-picked group of specialists assembled around the specific shape of the problem. Pavel Rapoport is founder and engineering anchor; the network around him is part of the offering.

Every product the studio builds is treated as a **platform with AI as the orchestration substrate** — not a product with AI features bolted on. This is a thesis, not a slogan: it shapes architecture decisions (e.g. AI is embedded in the runtime, not a sidecar), data decisions (a single source of truth per project, not silos), and operating decisions (per-client Telegram group, not ticket queues).

A new sub-product (e.g. the psychology product) is therefore expected to:

- Adopt the studio's stack unless it has a documented reason not to.
- Adopt the studio's operating model: Linear-tracked work, OpenSpec proposals, Forge-executed PRs, per-client Telegram group.
- Live in its **own repository**, not inside the platform monorepo. Code namespace and OpenSpec corpus are independent.
- Reference the platform repo (`openspec/current/platform.md`) as upstream context but **not import code from it**. Sub-products are downstream consumers in business terms only; they share no runtime artifacts with the platform repo.

---

## 2. Operating model — how a project moves

```
1. Lead arrives  →  studio inbox (web /contact form, referral, direct)
2. Pavel reviews →  Accept opens a Project (Linear project + Telegram group + canvas)
3. Project shaping (curation)
     ├─ Pavel selects collaborators
     ├─ Brief in Telegram group
     └─ Notes/decisions flow → /note → Linear backlog
4. Specs and execution
     ├─ Linear issue + linked OpenSpec proposal (in this repo's openspec/changes/)
     ├─ Forge generates PR from approved proposal
     └─ Human review → merge → deploy
5. Delivery
     ├─ Project marked delivered
     ├─ Case study draft (with team credits)
     └─ Telegram group archived or kept for ongoing relationship
```

Two non-negotiables for any sub-product:

- **Linear is the work tracker.** Every meaningful change has a Linear issue with a stable ID. The agent that executes the change is invoked **with that ID** and works against it.
- **OpenSpec is the source of truth.** Code that contradicts the spec is a regression. If the task contradicts the spec, the agent stops and surfaces the conflict — it does not silently choose.

---

## 3. Canonical stack

These are the studio defaults. A new sub-product inherits them unless it documents a deviation in its `platform.md § Decisions captured`.

| Layer | Tech | Version pin / notes |
|---|---|---|
| **App framework** | Next.js | 16 (App Router) |
| **UI runtime** | React | 19 |
| **Language** | TypeScript | 5 (strict) |
| **Styling** | Tailwind CSS | 4 |
| **Auth** | Supabase Auth | magic-link primary; OAuth case-by-case |
| **Database** | Supabase PostgreSQL | one project per product, RLS-first |
| **Storage** | Supabase Storage | use signed URLs; never public buckets for client data |
| **Build/runtime** | Cloudflare Workers via `@opennextjs/cloudflare` | not `@cloudflare/next-on-pages` (that one only supports Next ≤ 15.5.2) |
| **CI/CD** | GitHub Actions | secrets injected via Infisical |
| **Secrets** | Infisical | org "Rapoport Studio SLR", instance `eu.infisical.com` |
| **AI** | Anthropic Claude | Sonnet 4.x for routine, Opus for deep work / verification |
| **Project tracking** | Linear | each product gets its own team prefix (e.g. platform = `RAP-`) |
| **Code hosting** | GitHub | org `rapoport-studio`, private repo per product |
| **Client comms** | Telegram Bot API (primary), WhatsApp Cloud API (secondary) | Telegram first because Moldova reality + free + bots-in-groups |
| **Bridges** | n8n Cloud | Telegram/WhatsApp → studio API; never poll vendor APIs from app |
| **Package manager** | pnpm | workspaces, never npm/yarn |
| **Monorepo orchestrator** | Turborepo | pipeline in `turbo.json` |

Rationale snippets worth preserving when seeding a new platform.md:

- **Cloudflare Workers + OpenNext.** Secrets are read via `getCloudflareContext().env`, not `process.env`. The Anthropic SDK is the exception (it handles its own env). `opennextjs-cloudflare deploy` strips secrets on every deploy — `wrangler secret put` must be reapplied to every worker after deploy.
- **Cloudflare API token** must include `Workers KV Storage Edit` and `Workers Routes Edit` for OpenNext deploys to succeed.
- **Supabase service-role** is loaded via `getCloudflareContext().env`. Never embed it client-side.
- **Claude responses** sometimes wrap JSON in markdown fences — strip ` ```json ` before `JSON.parse`.
- **Linear SDK.** `issueSearch()` is deprecated, use `linear.issues({ filter })`.

---

## 4. Repository conventions

A sub-product gets a single private repo at `github.com/rapoport-studio/<product-slug>`. Inside:

```
<product-slug>/
├── apps/                    one or more Next.js apps, each deployed as a Worker
│   └── web/                 the primary app (rename per product, e.g. apps/practice/)
├── packages/                shared code; one per concern, never multi-purpose
│   ├── ui/                  design system (tokens + primitives + blocks)
│   ├── db/                  Supabase clients, types, migrations
│   ├── auth/                magic-link sessions + middleware proxy
│   ├── config/              typed env + per-vendor accessors
│   └── i18n/                locales (en + product's primary local language)
├── tools/                   scripts (deploy, secrets sync, migration drift checks)
├── supabase/migrations/     versioned SQL migrations
├── openspec/                OpenSpec corpus — see § 5
│   ├── current/             active capability specs
│   ├── changes/             in-flight proposals
│   ├── archive/             applied (dated YYYY-MM-DD-<slug>/)
│   └── AGENTS.md            agent guide for this corpus
├── package.json             root workspaces declaration
├── pnpm-workspace.yaml
├── turbo.json
├── tsconfig.base.json
├── eslint.config.mjs        ESLint flat config; package-boundary rule enforced
├── CLAUDE.md                operating manual for AI agents in this repo
├── README.md
└── ROADMAP.md               active phase tracker (one screen, not a backlog dump)
```

**Package namespace.** Use the full form `@rapoport-studio/*` (e.g. `@rapoport-studio/ui`). Short forms are not used. The platform repo's namespace is `@rapoport-studio/*` and sub-products may either share it (preferred when packages are intended to be lifted upstream one day) or use a product-scoped namespace `@rapoport-studio-<slug>/*` (when the product is unambiguously private).

**Package boundaries.** Encode the import graph in ESLint and treat violations as CI failures. The platform repo's table is canonical; for a new product, decide the boundaries in `platform.md` and lock them in `eslint.config.mjs`. Two rules that always hold:

- `apps/*` never import from each other.
- A package without a documented inbound edge has no consumers and probably should not exist.

**Dependencies.** Search the existing tree before adding any new dep. If you add one, document **why** in the PR description. Default answer is "we already have a way to do this".

---

## 5. OpenSpec methodology

The studio uses a **prose-style** OpenSpec convention. This is the single most important convention to seed correctly in a new project, because once a corpus drifts to a different shape, every change after that diverges further.

### 5.1 Folder shape

```
openspec/
├── current/                       capability specs — the source of truth
│   ├── platform.md                ← every project has one of these
│   ├── <capability>.md            ← one per capability
│   └── ...
├── changes/                       in-flight proposals (no date prefix)
│   └── <slug>/
│       ├── proposal.md            Why + What changes
│       ├── design.md              How (interfaces, data shapes, boundaries)
│       └── tasks.md               Phased checklist for the executor
├── archive/                       applied changes (date-prefixed)
│   └── YYYY-MM-DD-<slug>/
│       ├── proposal.md
│       ├── design.md
│       └── tasks.md
├── _methodology/                  cross-project reference material (this folder)
└── AGENTS.md                      agent guide for this corpus
```

### 5.2 Three-file proposal contract

Every change in `openspec/changes/<slug>/` has exactly three files: `proposal.md`, `design.md`, `tasks.md`. **There is no `specs/` directory, no per-capability delta files, no `Requirement` blocks.** Capability specs in `openspec/current/` are amended **in place at archive time**, not delta-restructured during the proposal.

### 5.3 What gets amended where, and when

- **During the proposal/implementation:** files only exist under `openspec/changes/<slug>/`. `openspec/current/` is untouched.
- **At archive (the very last phase of the change):** the executor appends a dated section to the relevant `openspec/current/<capability>.md` file using this exact pattern:

  ```
  ## <Title> (applied YYYY-MM-DD from change <slug>)
  ```

  Then renames `openspec/changes/<slug>/` → `openspec/archive/YYYY-MM-DD-<slug>/`. The OpenSpec CLI rejects date-prefixed change names with `--json`; manual `mv` is the workaround.

- **`openspec/current/` is otherwise read-only.** The only exception is typo / broken-link fixes, called out separately in the PR description.

### 5.4 Why we don't run `openspec validate`

The OpenSpec CLI's `openspec validate <change>` exits 1 on every studio change with the strict-deltas error ("Change must have at least one delta… ensure your change has a `specs/` directory"). The studio convention deliberately doesn't use that structure. **Skip the command.** Nothing in CI runs it. Nothing in Forge invokes it. If a contributor or tool reports the strict-deltas error, point them at `openspec/AGENTS.md`, not at restructuring the proposal.

### 5.5 The exemplar pattern

There is intentionally **no `TEMPLATE/` folder**. Templates go stale. The newest entry under `openspec/archive/` is the canonical example — copy its shape. For seeding a new project, the platform repo's `openspec/archive/2026-05-04-add-client-onboarding/` and `openspec/archive/2026-05-03-add-muse-research-tools-with-personas/` are good representative examples.

### 5.6 What `platform.md` must contain

A new project's `platform.md` must include, in roughly this order:

1. **What it is** — one paragraph for the product.
2. **Vision** (one sentence per layer).
3. **Brand boundaries** (what this product is and isn't, who it speaks to).
4. **Repository** layout (mirror § 4 of this charter, adapted).
5. **Domain & URL architecture** (every route, with v1/v2 status).
6. **Capability map** with a status legend (see § 8).
7. **Stack** (mirror § 3, with deviations).
8. **Operating model** (mirror § 2, adapted for the product's intake flow).
9. **Decisions captured** (numbered: D1, D2, … with rationale).
10. **Out of scope** (explicit list — what is intentionally not in v1).
11. **Glossary** (every product-specific term, defined once).

If you can't fill a section honestly yet, write a stub paragraph explaining what's missing and what input is needed. Do not invent.

### 5.7 Capability specs

Each capability gets its own `openspec/current/<capability>.md`. A typical product has, at minimum:

- `platform.md` (always)
- `auth.md`
- `db.md`
- `config.md`
- `ui.md` and `design-system.md` (split: `ui` is implementation, `design-system` is intent)
- `i18n.md`
- One or more product-specific capability specs (e.g. for the psychology product, a `practice.md` for the practitioner workspace, a `client.md` for the client-facing surface, a `sessions.md` for session lifecycle).

Each capability spec carries its own status (Active / Experimental / Stub / Deferred / Deprecated / TBD — see § 8).

---

## 6. Forge workflow

Forge is the studio's CLI executor (`packages/forge` in the platform repo). It turns Linear-tracked OpenSpec proposals into pull requests via Claude. A new sub-product **does not vendor a copy of Forge**. Instead, it consumes Forge from the platform repo by:

- Pinning the same Anthropic + Linear + Supabase identities (see § 9).
- Mirroring Forge's six command surfaces: `audit`, `spec`, `review`, `estimate`, `render`, `mirror-export`. Forge stays project-agnostic; the new product configures it via `forge.config.{mjs,js,json}` at its repo root.
- Treating `forge audit <module>`, `forge review`, and `forge spec <module> --fix` as the three commands an agent or human will reach for most often.

A new product's agent should be assumed to operate in the standard execution loop:

```
1. Read assigned Linear issue → has tasks + checkpoint
2. Read linked OpenSpec change in openspec/changes/<slug>/
3. Read affected capability specs in openspec/current/
4. Create branch:  forge/<change-id>/<task-slug>
5. Execute the tasks
6. Run lint, typecheck, tests
7. Open PR — title and body reference the Linear issue
8. Update Linear: comment with PR link, leave status to human reviewer
```

---

## 7. Naming conventions

### Branches

```
forge/<change-id>/<task-slug>

Examples:
forge/add-practice-foundation/scaffold-monorepo
forge/add-session-lifecycle/draft-rls-policies
```

### Commits

```
forge(<change-id>): <task title>

Examples:
forge(add-practice-foundation): add root package.json with workspaces
forge(add-session-lifecycle): port RLS helpers from platform
```

### PR title

```
<Linear-ID>: <issue title>

Example:
PSY-12: Phase 1 — Repo creation & scaffolding
```

### PR body template

```markdown
Closes <Linear-ID>

## Summary
<one paragraph: what this PR does>

## Changes
<bullet list of significant changes>

## Linked spec
- Change: `openspec/changes/<change-id>/`
- Affects: <capability specs touched>

## Checkpoint verified
<list checkpoint items from Linear issue, mark which are now satisfied>

## Notes for reviewer
<anything that needs human attention>
```

---

## 8. Status legend (capability specs)

| Status | Meaning |
|---|---|
| **Active** | Spec mirrors current implementation and is reviewable as design source-of-truth. |
| **Experimental** | Implementation exists but has known stabilization blockers; not yet production-stable. |
| **Stub** | Spec fixes surface area and core flows but defers detailed design to a later change. |
| **Deferred** | Vision is fixed; implementation is intentionally not in flight. |
| **Deprecated** | Vision was retired in favor of a successor capability. Spec retained as a tombstone redirect. |
| **TBD** | Capability acknowledged in the map but no spec file yet. |

Every capability in a new product's capability map carries one of these statuses. Honesty matters: a half-implemented surface is `Experimental` or `Stub`, not `Active`.

---

## 9. Hard rules

These are non-negotiable across every studio repository.

1. **Specs are the source of truth, not your assumptions.** If a task contradicts a current spec, stop and surface the conflict.
2. **Do not modify files outside task scope.** If you discover something else broken, note it in the PR description; don't fix it in the same PR.
3. **Package boundaries are enforced by ESLint.** Don't add deps the boundary table doesn't allow.
4. **Do not add dependencies without justification.** Default answer is "we already have a way to do this".
5. **Do not skip tests / verification.** A checkpoint that says "X works" requires evidence X actually works.
6. **Do not touch secrets.** Secrets live in Infisical and are applied via `wrangler secret put`. If a task needs a new secret, add the **name** in code, never the value.
7. **Do not modify `openspec/current/` directly** outside an archive step (see § 5.3).
8. **If blocked, stop and explain.** Open the PR (or comment on the issue) with what's done, what's blocked, and what input you need. Do not improvise into a wrong direction.

---

## 10. Identities & secrets

The studio operates programmatic actors under three dedicated bot identities, distinct from any human's personal credentials:

| Identity | Purpose | Infisical secret |
|---|---|---|
| **GitHub** | `rapoport-studio-bot` (org Member, fine-grained PAT, 90-day expiry, `Contents` / `Pull requests` / `Issues` read-write only — **no `Administration`**) | `GITHUB_BOT_TOKEN` |
| **Linear** | `hello@rapoport.studio` workspace Member personal API key, default scope | `LINEAR_BOT_API_KEY` |
| **Supabase** | Project service role; consumed in code via `getCloudflareContext().env` | `SUPABASE_SERVICE_ROLE_KEY` |

A new sub-product gets its own Supabase project and its own service-role key. GitHub and Linear identities are shared studio-wide.

**Local development** uses Pavel's personal PATs, not the bot identities. Bot identities are production-only.

**Verification.** Adapt `tools/verify-bot-tokens.ts` from the platform repo: a tiny script that probes GitHub `/user`, Linear `viewer`, and Supabase REST. Three `[ok]` lines and exit 0 mean the identity surface is healthy.

**Rotation.** GitHub PAT rotates every 90 days (scheduled). Linear and Supabase keys rotate on exposure only.

---

## 11. Client communication architecture

> Section rewritten 2026-05-09 from change `add-tg-canvas-surface` (RAP-121). The pre-2026-05-09 version described a single per-client TG group with `@rapoport_studio_bot` running slash-commands; that model is retired in favour of a two-bot taxonomy.

Telegram surfaces operate at two scopes — a personal-journal scope (Pavel) and a per-canvas team scope (owner + collaborators). A new sub-product inherits both shapes verbatim.

### 11.1 Pavel-bot — `@rapoport_studio_bot` (Digital Pavel)

Endpoint: `/api/studio/command`. Standalone Claude agent — **not Muse**. Routes to Pavel's personal Linear journal. Trigger surface (canonical):

| Trigger | Action |
|---|---|
| `/note <text>` | Creates Linear issue in Pavel's personal project, status = backlog |
| `/task @user <text>` | Creates Linear issue with assignee |
| `/done <ID>` | Closes the referenced issue |
| `/status` | Bot replies with project summary (open issues, in-progress, due) |
| `/search <query>` | Searches Linear, replies with top matches |
| 📝 reaction on a message | Same as `/note` (uses the message text) |
| ✅ reaction on a message linked to an issue | Marks issue as done |
| `@rapoport_studio_bot <free text>` | Claude interprets intent, decides action |

This bot is **scoped to Pavel** (single user, single Linear team). It is not the canvas-team work surface. New sub-products inherit this trigger surface for their own founder-as-PM journaling — re-defining `/note` semantics per product is a footgun.

### 11.2 Canvas-bot — `@rs_canvas_bot` (Muse, headless)

Endpoint: `/api/studio/telegram/webhook`. Each canvas with engagement opt-in (`projects.tg_surface_enabled = true`) gets its own TG group. Members: canvas owner + collaborators (clients out of scope in v1, see `add-client-role`). Voice + text input parity. The bot runs **Muse headlessly** (`runMuseHeadless` — server-loaded contexts, no browser) and gates all mutations through propose-mode approvals from the canvas owner (`/yes` confirm).

Read tools (Linear queries, GitHub reads, web research, OpenSpec readers) execute directly. Mutator tools (Mirror writes, GitHub writes, Architect emit) write to `canvas_proposals` and require owner `/yes`.

Engagement-level opt-in is recorded under [`rapoport-studio-engagement.md § 8 Appendix B → D11`](rapoport-studio-engagement.md). Default off; explicit acknowledgement of GDPR special-category ban (Art. 9), voice retention (R2 30-day lifecycle), and propose-only mutation semantics required before opt-in.

Capability spec: [`openspec/current/telegram.md`](../current/telegram.md).

### 11.3 Identity binding

Owner generates a 24h one-shot JWT invite token in canvas UI; TG user pastes it in DM with `@rs_canvas_bot` to bind their Telegram identity to their studio account (`tg_user_bindings`, `tg_invite_tokens`).

### 11.4 Other surfaces

WhatsApp setup remains for clients who prefer it; n8n normalizes payloads to a single shape so route handlers don't care about source. Public TG channel + discussion group are planned future surfaces, not in this change.

---

## 12. Glossary

| Term | Meaning |
|---|---|
| **Studio** | The company (Rapoport Studio SRL). Also: the platform's internal workspace at `app.rapoport.studio`. |
| **Sub-product** | A separately-repoed product the studio runs (e.g. the psychology product). Lives in its own GitHub repo with its own OpenSpec corpus. |
| **Forge** | The CLI in the platform repo's `packages/forge`. Turns OpenSpec proposals + Linear issues into PRs via Claude. |
| **Capability** | A first-class concern with its own OpenSpec file. Each capability evolves independently. |
| **Change** | A proposed modification to one or more capabilities, with `proposal.md` / `design.md` / `tasks.md`. |
| **Curation** | The studio's operating model: hand-picked team per project, not fixed staff. |
| **AI orchestration** | The thesis that AI is the architecture layer of a platform, not a feature bolted on. |

---

## 13. What the agent should produce next

After reading this document and the project's discovery material, the agent should:

1. **Create `openspec/AGENTS.md`** — copy the platform repo's version verbatim, adjust only the prefix references (Linear team key, etc.).
2. **Create `openspec/current/platform.md`** for the new product — using § 5.6 of this charter as the structural checklist. Honesty over completeness: stub sections that need human input.
3. **Create capability stubs** in `openspec/current/` — one per concern named in `platform.md § Capability map`. Status `Stub` is fine for v0.
4. **Create `CLAUDE.md` at the repo root** — operating manual for agents in the new repo, structured like the platform repo's `CLAUDE.md` (workflow, hard rules, environment gotchas).
5. **Create `ROADMAP.md`** — one screen, current phase only, no backlog dump.
6. **Open one Linear issue** for "Phase 0: bootstrap OpenSpec corpus" and attach the agent's output as a single PR. Subsequent work proceeds via the standard Forge loop (§ 6).

If anything in this charter is unclear, ambiguous, or doesn't fit the new product's reality — **say so in the bootstrap PR description**. Specs evolve; this charter is the current snapshot, not the final word.

---

## Agents and philosophy (applied 2026-05-08 from change `add-philosophy-foundation`)

Studio operates as an orchestra of six agents — Maestro, Muse, Atlas, Echo, Pulse, Forge — coordinated by Maestro and composed by Pavel. The roster, roles, and storage ownership are defined in [`openspec/current/agents-overview.md`](../current/agents-overview.md), which is the single source of truth.

Studio philosophy rests on four commitments — to craft, to awakening, to choice, to lineage. The full statement lives in [`openspec/_methodology/manifesto.md`](manifesto.md) (internal-first; public excerpts curated by Pavel from the same source). An internal Cast Club 27 layer pairs each agent with a second name; that layer is team-only and never appears in public branding, client deliverables, or marketing copy.

Layer 0 of the foundation (this amendment + `agents-overview.md`) ships ahead of the manifesto file itself — `manifesto.md` lands in Phase B of the same change, blocked on Pavel-voice authorship. Until then, the link above resolves to a missing file by design; it is the marker of incomplete foundation.

---

## TG canvas surface added to platform charter (applied 2026-05-09 from change add-tg-canvas-surface)

This charter previously described a single per-client TG group hosted by `@rapoport_studio_bot` running slash-commands. That model is retired. § 11 (Client communication architecture) was rewritten in place to describe the two-bot taxonomy:

- **Pavel-bot** (`@rapoport_studio_bot`) — Digital Pavel's personal Linear journal at `/api/studio/command`. Slash-command surface unchanged. Scoped to Pavel as a single user. **Not Muse.**
- **Canvas-bot** (`@rs_canvas_bot`) — per-canvas headless Muse at `/api/studio/telegram/webhook`. Voice + text parity. Owner + collaborators only (clients out-of-scope v1). Propose-only mutations gated on owner `/yes`. Default off; engagement opt-in via `projects.tg_surface_enabled`.

### What's gone (vs pre-2026-05-09 charter)

- Per-CLIENT TG group narrative ("one group per client, hosted by `@rapoport_studio_bot`").
- Slash-commands `/note`, `/task`, `/done` characterised as the canvas-team workflow — they remain only on Pavel-bot.
- Implicit single-bot framing.

### What's new

- Two-bot taxonomy with disjoint endpoints, secret tokens, and Infisical secrets.
- `runMuseHeadless` runtime contract (browser-independent Muse).
- Propose-only mutation semantics (`MUTATOR_TOOLS` allowlist; `canvas_proposals` table).
- JWT-based TG identity binding (24h one-shot, `tg_invite_tokens` + `tg_user_bindings`).
- Engagement opt-in flag (D-TCS-11) recorded under `rapoport-studio-engagement.md § 8 Appendix B → D11`.
- GDPR Art. 9 special-category ban on the canvas-bot surface (worker classification rejects + does not store).
- Daily + lifetime cost caps (`canvas_daily_costs`, `canvas_sessions.daily_cap_cents`); fail-closed.

### Cross-references

- Capability spec: [`openspec/current/telegram.md`](../current/telegram.md) — inaugural file.
- Engagement methodology: [`openspec/_methodology/rapoport-studio-engagement.md § 8 Appendix B → D11`](rapoport-studio-engagement.md).
- Studio API surface: `openspec/current/studio.md § /api/studio/telegram/* surface`.
- Database surface: `openspec/current/db.md § TG canvas surface schema`.
- Muse runtime: `openspec/current/muse.md § TG canvas surface as Muse — Headless mode`.
- Platform-level integration: `openspec/current/platform.md § Client communication architecture` (rewritten in same change).

A new sub-product inherits both bot shapes — Pavel-bot for founder journaling, canvas-bot for per-canvas team workflow — but provisioning canvas-bot per-product is **not** required for v0 of a sub-product. Add it once that product has its own Linear team + Supabase project + a canvas with non-trivial team activity.
