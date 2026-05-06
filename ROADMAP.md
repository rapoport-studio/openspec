# Roadmap — Studio platform v1 → v2

This roadmap is the master sequencing document for upcoming changes after the May 4 2026 design conversation. It consolidates the per-change roadmaps that previously sat in `add-projects-table/roadmap.md` and replaces ad-hoc deferral lists in individual change folders. New changes that come up in design conversations should update this document first, then spawn their own change folder.

---

## Vision recap

The studio operates around three nested levels:

1. **Studio** — the workshop itself. Methodology, spec library, two GitHub identities (`paveliko` for personal work, `rapoport-studio-bot` for programmatic operations once `add-github-bot-and-creds` lands).
2. **Project** — one body of work (client engagement, internal platform deliverable, external collaboration). Contains one or more Canvases. Optionally bound to a GitHub repository, a Linear team or project, and a Mirror perception layer. **Bindings are optional throughout the lifecycle**, never required.
3. **Canvas** — a design board for an idea. Persistent, multi-view workspace. Inside a Canvas, work moves through four phases: **Capture → Research → Spec → Build**.

Today the studio runs primarily on the Spec phase (Architect mode is mature, multi-tool Muse capable of authoring proposal/design/tasks triplets). Capture and Research are partial. Build is manual.

The roadmap closes these gaps without inflating any single change.

---

## Sequencing principle

Each change must be reviewable in one session by one reviewer. That keeps the discipline for narrow scope and forces correct dependency ordering. Where two changes are independent, they can ship in parallel.

---

## Active and immediate (May 2026)

These three changes are the top of the queue. The first two are in proposal-ready state with this conversation; the third is independent and can ship anytime.

### `add-projects-table` *(shipped 2026-05-04 — RAP-43)*

First-class `projects` table, FK from `canvas_sessions`, RLS mirroring sibling tables, two-row backfill of existing distinct slugs (`i-avocat`, `test`). No application code changes. Schema permits all integration columns nullable forever. Archived at `openspec/archive/2026-05-05-add-projects-table/`.

### `add-canvas-file-lifecycle` *(shipped 2026-05-04 — RAP-44)*

Replaced `canvas_artifacts` (versioned, three-name-only) with `canvas_files` (path-keyed, UPSERT). Added `mode='draft'|'project'|'legacy_promoted'` to `canvas_sessions`. Server actions `convertToProjectAction`, `saveFileAction`, `saveAllPendingAction`, `openPullRequestAction`, `copyFileAction` shipped. Workers fs gap for `write_mirror_field` closed. Archived at `openspec/archive/2026-05-05-add-canvas-file-lifecycle/`.

**Drift to track separately:** the design called for a `GitHubClient` in `packages/forge/src/core/`. Shipped reality has only a narrow `checkRepoExists` helper at `apps/studio/src/lib/github-client.ts` (introduced later by `add-project-bindings`). A full Octokit wrapper for branch/commit/PR operations does **not** exist anywhere — `RAP-48` Wave 3 (`add-forge-builder-mode`) remains GitHubClient-blocked despite RAP-44 closure. Resolution path TBD (move helper into `packages/forge` and expand, or build a separate forge-side client, or change RAP-48 design to use Octokit directly).

### `add-mirror-runtime-fix` *(shipped)*

Closed the `node:fs/promises` gap on Cloudflare Workers for `read_openspec` and `read_archive` by bundling `openspec/{current,archive}/**/*.md` into a generated TypeScript module embedded in the Worker at build time (Option B). Muse tools now select fs → bundle → `not_available_in_runtime`. `write_mirror_field` is handled separately inside `add-canvas-file-lifecycle`.

**Status:** Shipped via PR #171 (RAP-49). Archived at `openspec/archive/2026-05-05-add-mirror-runtime-fix/`.

---

## External project enablement (May 2026)

Studio-side engineering track that turns Forge from a single-monorepo executor into a cross-repo platform serving external engagements. Unbind (`rapoport-studio/unbind.life`) is the first dogfood; Vivod and Dentour follow.

This track must complete before any external Forge run. It does **not** block parallel internal work (homepage RAP-82 *(shipped 2026-05-06)*, canvas vertical RAP-78/79/80, conversation discipline RAP-52, community signup `add-studio-self-serve-signup` scaffold). Those tracks ship independently.

Trigger: Pavel + Olya kickoff resolves D-SE1 (per `openspec/_methodology/projects/unbind.yaml § engagement` and `unbind.life/openspec/changes/add-studio-engagement/`). The studio's preliminary `unbind.yaml` declares `mode: full`; the founders-agreement may downgrade to `deliverables-only`.

### Epic — `enable-unbind-as-first-forge-external-project` *(RAP-83, in flight)*

Six children, ordered as a critical path:

#### `add-project-connect-onebutton` *(RAP-84, scaffold open)*

GitHub App + install flow + bind UI in `apps/studio`. Replaces today's "Pavel pastes the repo identifier into a server action" with a one-click install path. Olya installs the App on `rapoport-studio/unbind.life`; install webhook fires; project record + bindings + credentials land automatically. Reusable for Vivod / Dentour.

Open questions on the scaffold ack: GitHub App vs OAuth App, App ownership (org vs user), credential storage (Supabase vs Infisical), token-mode handoff to Forge (per-installation JWT vs long-lived PAT), interaction with existing `add-project-bindings` (RAP-55/56).

#### `add-forge-multi-project` *(RAP-85, scaffold open)*

Config schema upgrade. `forge.config.mjs` gains `projects: Record<slug, ProjectConfig>` with per-target `issuePrefix`, `repo`, `botAccount`, `worktreeRoot`, `conventionAdapter`. CLI flag `--project <slug>` plumbed through `inbox / inspect / plan / build / status / cancel`. Default-project resolution by `cwd` match. Backwards-compatible: today's studio-only config keeps working.

Open questions: config shape (flat vs hierarchical), worktree root location, bot identity per project, Linear team override, App tokens vs PAT.

#### `add-forge-convention-adapter` *(RAP-86, scaffold open)*

Per-project OpenSpec shape adapter. The studio uses prose-style minimal (3 files, no frontmatter, no registries). Unbind extends this with YAML frontmatter on capabilities, four cross-cutting registries (`decisions/dependencies/maturity/open-questions`), capability-prefixed decision IDs (`D-SE1`, `D-RP-1`), maturity scoring 0–5, and the diff-at-archive pattern (build agent describes diffs in `tasks.md`, never applies them to `current/`). Without this adapter, Forge's build agent breaks Unbind's contract.

Adapter interface ships built-in for `studio` + `unbind`. Generic enough to add future engagement adapters via `forge.config.mjs § projects.<slug>.conventionAdapter`. Implementation Wave 1 depends on `add-forge-multi-project` Wave 1.

#### `extend-rap63-backfill-unbind` *(RAP-87, direct impl)*

Adds `unbind` row to `apps/studio` `projects` table (UPSERT on slug), GitHub repo binding `rapoport-studio/unbind.life`, Linear team UUID for the **Unbind** team. Mirror placeholder at `openspec/mirror/unbind/perception.md`. Same idempotent-migration pattern as RAP-63's earlier backfill of `pavelrapoport-com`, `vivod`, `dentour`. Soft-depends on RAP-85 Wave 1.

#### `engagement-onboarding-runbook-unbind` *(RAP-88, ops/docs)*

Authors `openspec/_methodology/projects/unbind-kickoff-runbook.md` covering pre-kickoff, kickoff agenda, bot identity provisioning, repo collaboration grant, credentials handoff (all keys flow through Olya's email; Pavel is admin not owner), studio-side registration, Forge-config registration, first dogfood run, post-kickoff archive, off-ramp checklist. Generic `_template-kickoff-runbook.md` so Vivod / Dentour fork in <30 min.

#### `dogfood-forge-cross-repo-unb2` *(RAP-89, blocked on RAP-84/85/86/87 + RAP-88 executed)*

End-to-end validation. Picks **UNB-2** (`add-studio-engagement`) as the first cross-repo Forge run because it's documentation-only, the proposal+design+tasks are already authored on the Unbind side, it materializes the studio-engagement contract (unblocking every following Unbind change), and it exercises the convention adapter end-to-end. Post-mortem published at `openspec/_research/forge-cross-repo-first-run.md`.

#### `add-github-presence` *(RAP-90, in flight)*

Public trust surface that supports the rest of this epic. Two new public repos under the org: `rapoport-studio/.github` (org-profile `profile/README.md` rendering at [github.com/rapoport-studio](https://github.com/rapoport-studio)) and `rapoport-studio/openspec` (read-only mirror of `_methodology/` + `AGENTS.md` + `ROADMAP.md`, kept in sync via `.github/workflows/sync-openspec-mirror.yml`). The mirror is the public destination for the homepage trust pillar A link (RAP-82), the GitHub App description (RAP-84), and the engagement charter URL referenced from the kickoff runbook (RAP-88). Locked decision: `_methodology/projects/` is **not** mirrored — client identifiers stay private; the mirror carries the three core charters + universal templates only.

### Operational dependencies (not Linear issues)

Captured in the RAP-88 runbook, not as separate tickets:

- Pavel + Olya kickoff call → resolve D-SE1 + Q-SE-1..5
- Olya invites `rapoport-studio-bot` to `rapoport-studio/unbind.life` with `Contents: write` / `Pull requests: write` / `Issues: write`
- Provision Infisical workspace `unbind` under Olya's email (Pavel admin, Olya owner)
- Open Telegram group `Rapoport Studio · Unbind` (if D-SE5 confirmed)

### Subsequent engagements

Vivod and Dentour are pre-registered in the projects table (RAP-63) but have no engagement charter signed. Adding either is a copy-of-RAP-83 epic with a fresh kickoff runbook and (potentially) a different convention adapter. The infra built for Unbind is reused.

---

## Identities & credentials

### `add-github-bot-and-creds` *(shipped 2026-05-04 — RAP-47)*

Registered `rapoport-studio-bot` GitHub user under `hello@rapoport.studio`, invited to `rapoport-studio` org as Member, generated fine-grained PAT scoped to all org repos with Contents/PRs/Issues read-write but no Administration, stored in Infisical as `GITHUB_BOT_TOKEN`. Linear bot identity and Supabase service role setup ship together. Archived at `openspec/archive/2026-05-05-add-github-bot-and-creds/`.

This change separated bot identity from Pavel's PAT. Save and Convert actions can optionally execute under bot identity for attribution clarity. Identity rollout on existing canvases remains a separate concern (no change opened yet).

### `add-linear-bot-and-creds`

Rolled into `add-github-bot-and-creds` (RAP-47) per its consolidated scope. Linear bot member registration, `LINEAR_BOT_API_KEY` Infisical secret, and the verification probe all ship together with the GitHub bot work. No separate change folder.

---

## Muse capabilities — read-only first

### `add-muse-github-read-tools` *(shipped 2026-05-05 — RAP-53)*

Five read-only tools in `packages/muse`: `github_read_file`, `github_list_prs`, `github_get_pr`, `github_list_commits`, `github_list_branches`. Authenticated via the studio bot PAT (RAP-47), thin REST wrapper at `packages/muse/src/shared/github-client.ts` rather than `@octokit/rest` (~700 KB delta avoided). Archived at `openspec/archive/2026-05-05-add-muse-github-read-tools/`. Note: the prior ROADMAP draft mentioned `github_list_repos` / `github_get_repo` — replaced by the file-and-PR-centric surface that actually shipped.

### `add-muse-linear-read-tools` *(shipped 2026-05-05 — RAP-54)*

Four read-only tools in `packages/muse`: `linear_list_issues`, `linear_get_issue`, `linear_search`, `linear_list_cycles`. Thin GraphQL wrapper at `packages/muse/src/shared/linear-client.ts` (parallels the GitHub-side decision). Permission gate `['owner', 'collaborator']` — client role hard-denied. Archived at `openspec/archive/2026-05-05-add-muse-linear-read-tools/`.

---

## Project bindings

### `add-project-bindings` *(shipped 2026-05-05 — combines RAP-55 + RAP-56)*

Server-side verification + write surface for `projects.github_repo`, `projects.github_repo_url`, and `projects.linear_team_id`. Four `'use server'` actions in `apps/studio/src/app/actions/project-bindings.ts` — bind/unbind × GitHub/Linear — verifying via raw `fetch` against the GitHub and Linear APIs (using `GITHUB_BOT_TOKEN` / `LINEAR_BOT_API_KEY` from RAP-47) before writing. Linear UUID is always persisted (never the team key) so re-keying in Linear's UI does not break the binding. UI surface deferred to `add-project-home` (RAP-57). `linear_project_id` writer deferred to a follow-up — `add-project-bind-linear-project`. The "create new repo" path is deferred until `add-muse-github-write-tools` lands; same for "create new Linear project".

---

## Project home (UI)

### `add-project-home` *(shipped 2026-05-05 — RAP-57)*

Routes `/projects` (list) and `/projects/[slug]` (detail). Detail view shows recent canvases, bindings card (wraps the four `add-project-bindings` actions), and a knowledge-stats card aggregating `canvas_project_instructions` over `canvas_sessions`. Computed-only, no new tables.

**Follow-ups shipped same day:**

- `add-project-home-mirror-methodology` *(shipped 2026-05-05 — RAP-65, PRs #192 + #193)* — Mirror coverage scoreboard card + methodology card on `/projects/[slug]`, plus the build-snapshot infra (`tools/with-deploy-info.mjs`) that injects `DEPLOY_SHA` / `DEPLOY_AT` constants so both cards display "as of which deploy" honestly. Archived at `openspec/archive/2026-05-05-add-project-home-mirror-methodology/`.
- `fix-projects-rail-entry` *(shipped 2026-05-05 — RAP-68, PR #197)* — drift fix: rail navigation was missing the **P** entry; routes were reachable only via direct URL until this lightweight fix landed. Rail order is now D / C / P / I / T / S.

---

## Canvas as design board

### `add-canvas-multi-view` *(shipped 2026-05-05, Wave 1 only — registry primitive landed; switcher + Mirror view deferred to follow-up changes below)*

Surface the existing chat alongside additional views inside the canvas-view: Mirror viewer (perception fields rendered from canvas_files), file browser (`openspec/` subset), spec graph stub. Tabs or panels in the existing layout. No DB schema changes — these are renderings of existing state.

Wave 1 shipped a view registry primitive (`packages/canvas/src/views`) and extracted today's three-column body into `ChatView` — zero user-visible diff. Follow-up work split out:

- `add-canvas-view-switcher` — segmented control + localStorage persistence. Ships once a second view exists.
- `add-canvas-mirror-view` — Mirror viewer placeholder; ships once Mirror has enough perception field density to be worth a dedicated surface.

### `add-canvas-idea-capture`

First phase of canvas lifecycle. Structured input panel for problem statement, proposed solution, target audience, metadata (industry, deal size, timeline). Persisted in `canvas_sessions.wizard_state` JSONB column (already exists, currently underused). Muse references this state in subsequent phases.

### `add-muse-competitor-research`

Tool composition: `find_competitors(domain, audience)`, `analyze_competitor(url)`, `competitive_landscape(competitors[])`. Built from existing `web_search` + `fetch_url` + LLM extraction primitives. Renders comparison matrix as a canvas_files row under path `research/competitors.md`.

### `add-business-model-templates`

Knowledge base of business model patterns the studio reuses across engagements (SaaS, marketplace, freemium, transactional, agency). Storage TBD (markdown files in `openspec/templates/`, DB table, or skill-style files). Muse references during research and spec phases.

---

## Build phase automation

### `add-muse-github-write-tools` *(shipped 2026-05-05 — RAP-60)*

Five write tools: `github_create_branch`, `github_commit_file`, `github_open_pr`, `github_comment`, `github_request_review`. Each gates on explicit `approved: true` in input schema (`github_write_guard.ts` exporting `ensureWriteApproved`); without it, tools short-circuit with `approval_required`. v1 ships without DB-backed audit log (deferred). Archived at `openspec/archive/2026-05-05-add-muse-github-write-tools/`.

### `add-muse-linear-write-tools` *(RAP-61, scaffold drafted)*

Linear-side mirror of the GitHub-write surface. Four tools: `linear_create_issue`, `linear_update_issue`, `linear_add_comment`, `linear_archive_issue`. Same approval gate. Implementation gated on RAP-54 merge; scaffold proposes renaming `github_write_guard.ts` → `write_guard.ts` so the helper drops its service-specific naming.

### `add-forge-builder-mode` *(RAP-48, proposal ready)*

Forge as the platform architect: gains its own GitHub/Linear/Supabase identities and six new commands (`inbox`, `inspect`, `plan`, `build`, `status`, `cancel`). Reads a Linear issue + corresponding OpenSpec change, generates code via a Claude Agent SDK subprocess in a working tree, opens a PR, comments on the Linear issue with the PR link. Six explicit checkpoints; pre-merge checkpoint never auto-passes. v1.2 milestone in the studio roadmap. Hard prerequisite: RAP-47 must merge before Wave 3.

---

## Adoption

### `add-adopt-existing-projects` *(shipped 2026-05-05 — RAP-63)*

Three real projects backfilled into `projects` table: `pavelrapoport-com` → studio self-project (GitHub bound, Linear team `RAP`); `vivod` → Vivod client project (bindings deferred, NULL); `dentour` → Dentour collaboration (bindings deferred, NULL). Mirror placeholders created at `openspec/mirror/{pavelrapoport-com,vivod,dentour}/perception.md`. Migration is idempotent (UPSERT on slug); no hardcoded UUIDs (resolves owner from existing seeded `i-avocat` row, fails fast if seed missing). Slug uses `pavelrapoport-com` (not `pavelrapoport.com`) because the projects slug CHECK regex disallows dots; canonical name preserved. Archived at `openspec/archive/2026-05-05-add-adopt-existing-projects/`.

### `add-muse-scout-mode` *(scaffold shipped 2026-05-05 — RAP-64; v1.3 design lock)*

Forward-looking design lock for the Scout mode in v1.3 — codebase scanning produces an OpenSpec migration report. Scaffold captures six open architecture questions (Q-SC-1…6) with defaults: single-agent scan, sampling/async cost envelope, intermediate capability map output, read-only API access, no Mirror integration, persona model deferred. Hard prerequisites for v1.3 kickoff: v1 Architect dogfood mature (≥ 5 changes shipped), RAP-48 Wave 4 archived, ≥ 1 adopted project manually spec-authored, RAP-61 implementation on main. Scaffold archived at `openspec/archive/2026-05-05-add-muse-scout-mode/`; implementation chains open when v1.3 loop kicks off.

---

## Design system

### `add-entity-system` *(shipped 2026-05-06 — RAP-91)*

Catalogs ~38 primary domain entities across 7 tiers (platform-structural / domain / concept / network / content / spec / external). Locks Lucide as the canonical icon library with comparative analysis (Lucide / Phosphor / Heroicons / Tabler / Radix Icons) and recorded re-evaluation triggers. Defines the `EntityCard` primitive contract — four variants (`inline` / `row` / `card` / `detail`), fixed slot layout, responsive collapse, accessibility, i18n key shape. Five fixes to first-draft icon picks landed during init (`Seedling` → `GraduationCap` since `Seedling` is not in `lucide-react`; plus `Compass` / `GitPullRequestArrow` / `KanbanSquare` collisions or wrong Lucide names). Three icons remain held open for future ratification (`muse` `Wand2` vs `Atom`; `forge` `Hammer` vs `Anvil`; `community-member` `Sprout` vs `Leaf` — defaults shipped). Specs only — no application code. Archived at `openspec/archive/2026-05-06-add-entity-system/`. New capability spec at `openspec/current/entities.md`. Amendments applied to `design-system.md`, `platform.md`, `ui.md`.

**Follow-ups:**

- `add-entity-card-primitive` *(RAP-92, branch scaffolded)* — TypeScript registry mirroring the catalog + `<EntityCard>` primitive in `packages/ui/src/components/entity-card.tsx` + i18n placeholder JSON in `packages/i18n/messages/{en,ru,ro}/entities.json` + extension of the `i18n` smoke parity test to cover `entities.json`. No consumer migrations.
- `migrate-surfaces-to-entity-card` *(RAP-105)* — consumer migration sweep, one PR per consumer family: inbox rows, canvas list rows, project home Bindings card, file tree rows, work cards, network cards. Blocked by RAP-92.

---

## Independent behavioural changes

### `add-muse-conversation-discipline`

System prompt change covering three behaviours that emerged in May 4 2026 design conversations:

1. *Closing pulse* — at the end of meaningful turns, Muse outputs a "captured this turn" block (1-3 bullets of what landed) and a "suggested next moves" block (2-3 options with rough cost). Skipped on trivial turns.

2. *Continuous documentation reflex* — within each turn, Muse attempts to extract structure from speech: new concepts → potential spec sections, new constraints → potential Mirror fields, new commitments → potential proposal updates. Surfaces these as part of closing pulse.

3. *Role-aware questioning* — different question playbooks for different audiences. To `platform_owner`: strategic, risk-focused, what's-out-of-scope. To `collaborator`: technical, edge-cases, dependencies. To `client`: domain, current process, pain points. Muse picks the playbook based on the current message author's role.

This is one big system-prompt change, ~150-200 lines of buildSystemPrompt updates plus tests. Independent of project layer — ships whenever ready.

### `add-canvas-voice-input` *(scaffold shipped 2026-05-05 — RAP-51)*

Browser Web Speech API mic button next to the chat composer's send button — opt-in, append-not-replace transcript semantics, mobile-first ergonomics, no audio storage. Scaffold locks Web Speech only for v1; Whisper / Deepgram deferred until quality becomes the bottleneck. Archived at `openspec/archive/2026-05-05-add-canvas-voice-input/`; implementation ships separately.

### `add-canvas-and-session-split` *(deferred terminology)*

Today's `canvas_sessions` table mixes two concepts that the design conversation of May 4 2026 separated:

- **Canvas** — the persistent design board for an idea / topic / change. Multi-view workspace. Owns name, slug, status, files, artifacts, bindings.
- **Session** — a time-bounded sitting in a Canvas. Owns participants for that sitting, cost, message stream, started/ended timestamps.

Today, one `canvas_sessions` row plays both roles. The split is deferred until the multi-view Canvas is built and the right shape for Sessions is clear from real usage. At that point the rename touches DB tables, ~30 TypeScript files, routes, and Linear references — it's expensive, so it ships when value is concrete.

---

## Sequencing diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│  Active / immediate                                                  │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   add-projects-table          (shipped — RAP-43)                    │
│   add-canvas-file-lifecycle   (shipped — RAP-44)                    │
│   add-mirror-runtime-fix      (shipped — RAP-49)                    │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│  External project enablement (epic RAP-83 — in flight)               │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   kickoff (Pavel + Olya)  ──►  D-SE1 = full / deliverables-only     │
│                                       │                              │
│                                       ▼                              │
│   add-project-connect-onebutton (RAP-84, scaffold)                  │
│   add-forge-multi-project       (RAP-85, scaffold)                  │
│   add-forge-convention-adapter  (RAP-86, scaffold)                  │
│   extend-rap63-backfill-unbind  (RAP-87, direct impl)               │
│   engagement-runbook-unbind     (RAP-88, ops/docs)                  │
│                       │                                              │
│                       ▼                                              │
│   dogfood-forge-cross-repo-unb2 (RAP-89, blocked)                   │
│                       │                                              │
│                       ▼                                              │
│   first PR in unbind.life from rapoport-studio-bot                  │
│                                                                      │
│   add-github-presence           (RAP-90, in flight, parallel)       │
│       └── github.com/rapoport-studio + /openspec mirror             │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│  Identities & credentials                                            │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   add-github-bot-and-creds       add-linear-bot-and-creds            │
│           │                             │                            │
│           ▼                             ▼                            │
│   add-muse-github-read-tools  ✓  add-muse-linear-read-tools  ✓       │
│           │                             │                            │
│           └──────────────┬──────────────┘                            │
│                          ▼                                           │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│  Project bindings & home                                             │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   add-project-bindings  ✓                                            │
│           │                                                          │
│           ▼                                                          │
│                    add-project-home  ✓                               │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│  Canvas as design board                                              │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   add-canvas-multi-view                                              │
│   add-canvas-idea-capture                                            │
│   add-muse-competitor-research                                       │
│   add-business-model-templates                                       │
│   add-canvas-voice-input  (scaffold ✓)                               │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│  Build phase automation                                              │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   add-muse-github-write-tools  ✓  add-muse-linear-write-tools  (scaffold ✓) │
│           │                             │                            │
│           └──────────────┬──────────────┘                            │
│                          ▼                                           │
│                  add-forge-builder-mode                              │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│  Adoption                                                            │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   add-adopt-existing-projects  ✓                                     │
│           │                                                          │
│           ▼                                                          │
│   add-muse-scout-mode  (scaffold ✓ — v1.3 design lock)               │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│  Independent behavioural / deferred terminology                      │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   add-muse-conversation-discipline   (independent)                   │
│   add-canvas-and-session-split       (deferred until multi-view)     │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Documentation drift items (not changes, just doc fixes)

These were spotted during pre-flight verification of `add-projects-table` v2 and `add-canvas-file-lifecycle` v2. Each is a small `docs/rap-NN/...` PR scoped to one file. None are in scope for either active change.

1. `db.md` lists 4 public tables; live database has 5 (`inbox` exists from `add-client-onboarding`).
2. `db.md` does not document `canvas_sessions.wizard_state jsonb NOT NULL DEFAULT '{}'::jsonb`.
3. `db.md` describes `canvas_artifacts` with pending/committed columns that **do not exist** in the live schema. The live schema is the versioned model `(canvas_id, file, version)`. This is the most critical drift — the v1 of `add-canvas-file-lifecycle` was written against the documented (incorrect) state. v2 corrects this.
4. `db.md` does not document the fourth `canvas_sessions.status` enum value `'archived'`.
5. `canvas_sessions_tool_cost_cents_check` constraint name still uses pre-rename `tool_cost_cents` term; column was renamed to `cost_cents`. Cosmetic.
6. Memory snapshot Supabase project ID `mtavnbjdgldttqdpwouo` is stale; live ID is `nifagnmgwoqkplegsicy`.
7. Linear team ID in memory `ffb2de5f-f0b7-476f-ad46-979be7844800` is stale; live ID is `c48bbbb1-9eba-477d-87d2-d082c506d885` (verified via Linear MCP).

Items 6 and 7 will be corrected via `memory_user_edits` post-merge.
