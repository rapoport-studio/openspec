# Roadmap — Studio platform v1 → v2

This roadmap is the master sequencing document. It is organised around **MVP milestones**, not feature areas: the top sections are what blocks the next reviewable cut of the platform; the bottom sections are durable history and deferred work.

**Last full sync:** 2026-05-09 (post-engagement-dashboard-archive — `add-control-engagement-dashboard` Phase 1 shipped via #389 + archived as `archive/2026-05-09-add-control-engagement-dashboard/`; two of five `/control` cards (Roadmap + Engagements) now `ready`. Earlier same day: RAP-92 drift item closed via retroactive `archive/2026-05-06-add-entity-card-primitive/` reconstruction; `add-project-migration-prompt` Phase 1 shipped via #381 + archived (template precondition for Vivod / Dentour Phase 5 cleared); `add-studio-control-room` Phase 1 archived. Track B Phase 6 archived 2026-05-08; `sync-forge-builder-spec` Phase 2 shipped via #351; **RAP-48 closed 2026-05-09** — Forge Builder shipped end-to-end; first real-issue dogfood = this change `sync-roadmap-post-rap48-close`).

**How to read this file:**

- **Section 1** is the working plan. The MVP v1 critical path lives here. Everything that is not in Section 1 is parallel, deferred, or shipped.
- **Section 2** is the immediate next milestone after MVP v1 (Unbind dogfood).
- **Section 3** is parallel work — active changes that do not block MVP v1 and can ship alongside or after.
- **Section 4** is deferred / future work — explicitly out of scope for MVP v1.x.
- **Section 5** is shipped foundation — kept brief because it is durable history, not a plan.
- **Section 6** is drift / cleanup items — small fixes that don't fit into any change folder.

---

## Vision recap

The studio operates around three nested levels:

1. **Studio** — the workshop itself. Methodology, spec library, two GitHub identities (`paveliko` for personal work, `rapoport-studio-bot` for programmatic operations).
2. **Project** — one body of work (client engagement, internal platform deliverable, external collaboration). Contains one or more Canvases. Optionally bound to a GitHub repository, a Linear team or project, and a Mirror perception layer. **Bindings are optional throughout the lifecycle**, never required.
3. **Canvas** — a design board for an idea. Persistent, multi-view workspace. Inside a Canvas, work moves through four phases: **Capture → Research → Spec → Build**.

The Spec phase is mature (Architect mode, multi-tool Muse, full convention adapter). Capture is complete (sticky-note idea board shipped via RAP-58; structured discovery wizard shipped end-to-end via `add-canvas-discovery-wizard`, archived 2026-05-08). Research is partial (competitor research not yet scaffolded). Build phase has the runtime substrate (Forge UI surface RAP-130 + `add-forge-discovery-surface` Track-C close-out, multi-project config RAP-85, convention adapter RAP-86); the executor (RAP-48 Builder) shipped end-to-end (Waves 1–3 + Wave 4 Phases A/B/D, RAP-48 closed 2026-05-09); first real-issue dogfood is *this change* `sync-roadmap-post-rap48-close` (Track D / Phase C of `sync-forge-builder-spec`).

**Sequencing principle:** each change must be reviewable in one session by one reviewer. Where two changes are independent, they ship in parallel.

---

## 1. MVP v1 — Studio Self-Drive (CRITICAL PATH)

**Goal of MVP v1:** Pavel can complete the full design → spec → build → ship loop on the studio's own work **entirely through the studio UI**, not the bare CLI. When a small RAP-* issue closes via that loop end-to-end, MVP v1 is done.

### The seven-step end-to-end flow

```
1. Canvas → Capture idea          (structured design doc input)
2. Architect mode → OpenSpec      (proposal + design + tasks)
3. Linear issue ↔ change folder   (linkage)
4. /tasks/[key] → Plan            (Forge plans, Pavel approves)
5. /tasks/[key] → Implement       (Forge builds, opens PR)
6. /tasks/[key] → Review          (Forge reviews, Pavel merges)
7. Loop continues                 (drift documented, retro logged)
```

### Status per step

| # | Step | State | Today | Gap to MVP |
|---|---|---|---|---|
| 1 | Canvas Capture | ✅ 100% | sticky-board (RAP-58) ✓ + wizard schema/actions/UI/Muse/spec-amend/archive (`add-canvas-discovery-wizard` ✓ via #359/#363/#365/#366/#369/#371 + this PR) | — |
| 2 | OpenSpec workflow | ✅ 95% | Architect mature, Muse multi-tool, convention adapter | Polish (Mirror view) |
| 3 | Linear ↔ OpenSpec linking | ✅ 90% | Linear bot, Muse write tools | Auto-create issue from change folder (manual today) |
| 4 | Forge UI — Plan / Review / PR | ✅ 90% | `/tasks/[key]`, DB, 15 actions, SSE, Phases 1/3/4 + Track-C discovery surface (F rail + `/tasks` index + project Forge card, all archived 2026-05-08) | Implement body lit by Wave 4 |
| 5 | Forge UI — Implement (Phase 2) | 🔴 20% | Stub "visible-but-disabled" | Builder runtime (RAP-48) lights up the body |
| 6 | Forge Builder Mode (CLI) | ✅ 100% | Waves 1–3 ✓ (PRs #161/#166/#186) + Wave 4 Phase A ✓ (#351) + Phase B ✓ (RAP-151 sandbox seed 2026-05-08) + Phase 3.A drift fixes via #378/#379/#382/#385/#387/#391/#394 — RAP-48 closed 2026-05-09 | — |
| 7 | Studio dogfood | 🟡 in-flight | `sync-roadmap-post-rap48-close` (this change) is Cycle 1 of the 3-cycle dogfood campaign | Cycles 2 + 3 + retro |

### Critical-path tracks

```
Track A — RAP-48 close-out via      All waves merged; RAP-48 closed         [████████████████████] ✅ shipped
          sync-forge-builder-spec   2026-05-09 (Phases A/B/D done; C =
                                    Track D below)
Track B — add-canvas-discovery-     All phases shipped + archived          [████████████████████] ✅ shipped
          wizard                                                              (archived 2026-05-08)
Track C — add-forge-discovery-      F-rail + /tasks index + project card   [████████████████████] ✅ shipped
          surface                                                             (archived 2026-05-08)
Track D — Studio dogfood E2E        Cycle 1 = sync-roadmap-post-rap48-      [████░░░░░░░░░░░░░░░░] in-flight
          (= sync-forge-builder-    close (this change). Cycles 2 + 3
          spec Phase 3)             follow.
```

#### Track A — `add-forge-builder-mode` (RAP-48, ✅ closed 2026-05-09)

Forge as platform architect: own GitHub/Linear/Supabase identities, six commands (`inbox` / `inspect` / `plan` / `build` shipped Waves 1–3; `status` / `cancel` pending Wave 4), Anthropic Agent SDK in working tree.

**Hard prerequisites — all met as of 2026-05-08:**

- ✅ RAP-47 — bot identities
- ✅ Canonical `GitHubClient` consolidated in `packages/forge/src/core/github-client.ts` (RAP-136 / RAP-140)
- ✅ `add-project-connections` shipped end-to-end (archived 2026-05-08 as `archive/2026-05-08-add-project-connections`; operational tail — production deploy, manual smoke, follow-up `drop-deprecated-projects-binding-columns`, RAP-111 close — tracked separately)

**Already shipped (PRs #161 / #166 / #186, 2026-05-04 → 2026-05-05):**

- Wave 1 — identity foundation (`loadForgeIdentity` → `core/auth.ts`), `forge inbox`, `forge inspect`, programmatic API on `Forge` handle.
- Wave 2 — `forge plan` + lessons-learned mechanism (`~/.forge/lessons-learned.md`) + state model (`~/.forge/state/*.json`) + Builder prompt template (`packages/muse/src/builder-prompt.ts`).
- Wave 3 Phase 8 — build orchestrator + Anthropic Agent SDK call shape (per spike `archive/2026-05-05-add-forge-builder-mode/spikes/2026-05-05-claude-agent-sdk.md`) + six-checkpoint gating.

**Wave 4 — fully shipped, tracked under `openspec/changes/sync-forge-builder-spec/`:**

- ✅ Phase A — `forge status` + `forge cancel` orchestrators + in-process `build-registry.ts` + `GitHubClient.deleteBranch` (merged via #351 on 2026-05-08).
- ✅ Phase B — sandbox lessons-seed dogfood completed via RAP-151 (2026-05-08); seed entry written to `~/.forge/lessons-learned.md`.
- 🟡 Phase C — first real-issue e2e in flight via Track D Cycle 1 (`sync-roadmap-post-rap48-close` — this change).
- ✅ Phase D — Phase 3.A drift fixes shipped via #378/#379/#382/#385/#387/#391/#394 between 2026-05-08 and 2026-05-09; RAP-48 closed 2026-05-09.

Note: a separate `openspec/changes/sync-forge-builder-spec/` change owns the documentation truthing (forge.md / muse.md / platform.md / ROADMAP) plus Wave 4 execution. RAP-48 was prematurely closed 2026-05-05 after Wave 3 Phase 8 merged; reopened to "In Progress" 2026-05-08 to absorb Wave 4 + truthing.

Estimated remaining effort: **complete** for RAP-48 itself. First real-issue e2e (Track D Cycle 1) is in flight via this change.

#### Track B — `add-canvas-discovery-wizard` ✅ shipped (archived 2026-05-08 as `archive/2026-05-08-add-canvas-discovery-wizard`)

Structured input panel for problem statement, proposed solution, target audience, metadata (industry, timeline, budget). Persisted in `canvas_sessions.wizard_state` (JSONB column already existed). Muse references this state via shared system-prompt preamble across Architect / Builder / Research modes.

> **Naming note:** the slug `add-canvas-idea-capture` is already taken by the shipped sticky-note board (RAP-58, archived 2026-05-05). The two changes are complementary: sticky-notes are spatial free capture; the wizard is sequential structured intake at canvas creation.

**Per-phase ship log:**

- Phase 0 — scaffold (#356)
- Phase 1 — Zod schema in `packages/canvas/src/wizard/` + 32 tests (#359)
- Phase 2 — server actions (`saveWizardField` / `clearWizardField` / `loadWizardState`) + 18 tests (#363)
- Phase 3 — UI panel `<DiscoveryWizard>` with six fields, autosave on blur, collapsibility per-canvas, read-only branch at `tasks` stage + 17 tests (#365 + #366)
- Phase 4 — Muse integration via `renderWizardBlock` in shared `buildSystemPrompt` (mode-agnostic) + 16 tests (#369)
- Phase 5 — spec amendments (`studio.md` + `muse.md` + `db.md` + `lifecycle.yaml` `wizard_field_min_chars` check kind) + 5 evaluator tests (#371)
- Phase 6 — archive (this PR)

**v1 deviations documented in archive:** tests for `packages/canvas/src/wizard/` schema live in `apps/studio/__tests__/canvas-wizard/` rather than the canvas package (no vitest infra in canvas yet); Phase 2 used read-then-merge-then-write for JSONB merge instead of raw `||` (PostgREST limitation); Phase 4 collapsed T4.1 + T4.2 into one wiring task (single shared `buildSystemPrompt`, not per-mode builders).

#### Track C — `add-forge-discovery-surface` ✅ shipped (archived 2026-05-08 as `archive/2026-05-08-add-forge-discovery-surface`)

Closed three gaps from RAP-130 explicitly out of scope at the time:

1. **F rail icon** — fixed via #343 (Phase 1); `studio-rail.tsx` now links to `/tasks`.
2. **`/tasks` index** — shipped via #345 (Phase 2); 50 most-recent `forge_runs` across all projects, server-rendered with empty-state, status pills, and per-row links to `/tasks/[key]`.
3. **Project-detail Forge card** — shipped via #347 (Phase 3); `<ForgeRunsCard>` on `/projects/[slug]` with two empty-state branches keyed on whether the project has a known Linear-issue prefix.

Spec amendments to `studio.md § Task discovery` + `forge.md § Out of scope` shipped via #348 (Phase 4). Track D (studio dogfood E2E) precondition is now unblocked.

**v1 deviation (documented in archived design.md § 3):** project columns deferred until a follow-up migration adds `forge_runs.project_id` (or `projects.linear_prefix`). Today the project card scopes by `linear_issue_key LIKE 'PREFIX-%'` against a static slug→prefix map mirroring `forge.config.mjs § projects`.

#### Track D — Studio dogfood E2E (no change folder, runbook only)

Once Tracks A and C land, take a small RAP issue (e.g., the F-rail fix from Track C if not yet merged, or a tiny UI polish) and run the full cycle:

```
forge inbox --project=studio
forge inspect RAP-XXX
forge plan RAP-XXX
forge build RAP-XXX
→ verify Linear comment + PR created
→ merge → archive change folder
→ post-mortem at openspec/_research/forge-studio-first-run.md
```

Estimated effort: **4–6 hours** + drift fixes for whatever breaks. **Definition of done for MVP v1.**

### Dependency graph

```
Track C (forge-discovery-surface)        ✅ done & archived 2026-05-08
Track B (canvas-discovery-wizard)        ✅ done & archived 2026-05-08
                                                      │
                                                      ▼ (D unblocked from B + C sides)

Track A (sync-forge-builder-spec)        🟡 Phase A ✓ — Phases B–C remain
   │                                         (Phase C ≡ Track D)
   ▼
Track D (Studio dogfood E2E)             baked into sync-forge-builder-spec Phase 3
                                                      │
                                                      ▼
                                              🎉 MVP v1 done — ETA 1–2 days
```

### Items deprioritised to keep MVP v1 fast

These are explicitly **not** required for MVP v1 and live in Section 3 or 4:

- `add-tg-canvas-surface` — Telegram is nice but not required for self-drive
- `add-network-public-surface` / `add-network-studio-surface` — Network is its own product
- `add-linear-oauth-connection` — bot identity works, OAuth is nice-to-have
- `add-canvas-view-switcher`, `add-canvas-mirror-view` — deferred until second view exists / Mirror density grows
- `add-project-connect-onebutton` Phase 2+ — gated on Olya kickoff (separate human dependency)

---

## 2. MVP v1.1 — Unbind dogfood (immediate next, after v1)

The Unbind external-engagement enablement (epic [RAP-83](https://linear.app/rapoport-studio-slr/issue/RAP-83)) is the milestone right after MVP v1. Its substrate is shipped; what remains is the executable tail and human-coordinated kickoff.

**Substrate shipped:**

- ✅ RAP-85 multi-project config
- ✅ RAP-86 convention adapter (full — Phase 6 archived)
- ✅ RAP-87 backfill `unbind` row + bindings + Mirror placeholder
- ✅ RAP-89 preflight (PR #300 — `unbind` in `forge.config.mjs`)
- ✅ RAP-130 Forge UI surface (Plan/Review/PR)

**Open (gated on MVP v1 Track A or human dependencies):**

- `migrate-unbind-into-studio` *(scaffold via PR #326, 0/43 done)* — formalises operating Unbind from inside the studio app once MVP v1 substrate is ready.
- `engagement-onboarding-runbook-unbind` *(RAP-88, in flight)* — `_methodology/projects/unbind-kickoff-runbook.md` + generic `_template-kickoff-runbook.md`.
- `add-github-presence` *(RAP-90, in flight)* — `rapoport-studio/.github` org-profile + `rapoport-studio/openspec` public mirror.
- `add-project-connect-onebutton` Phase 2+ *(RAP-84 / RAP-126, gated)* — GitHub App install flow, blocked on Olya kickoff.
- `dogfood-forge-cross-repo-unb2` *(RAP-89 full, blocked)* — first cross-repo Forge run against UNB-2. Blocks on RAP-84 + RAP-88 + RAP-90 + MVP v1 Track A.

**Subsequent engagements (deferred to late May / June):**

- `onboard-vivod` *(scaffold via PR #329)* — durable engagement record for Vivod.
- `onboard-dentour` *(scaffold via PR #324)* — Dentour kickoff plan.
- `add-project-migration-prompt` *(✅ shipped via #381, archived 2026-05-09 as `archive/2026-05-09-add-project-migration-prompt/`)* — generic Branch-B migration template authored at `_methodology/projects/_template-migration-prompt.md`. Template-precondition for Vivod + Dentour Phase 5 has cleared; per-engagement forks (`vivod-migration-prompt.md`, `dentour-migration-prompt.md`) can be authored when each engagement's remaining gates (charter / mode / `dialect_change_approved`) hold.

---

## 3. Parallel tracks (active, do not block MVP v1)

These are active change folders that **can ship in parallel** with MVP v1 work or right after. They are not on the critical path.

### Canvas vertical (Wave 2 leftovers)

- `add-xstate-state-orchestration` *(RAP-128 Wave 1 ✓ + RAP-133 Wave 2A ✓; Wave 2B in flight)* — broader XState orchestration beyond chat. ~88% checkbox completion.

### Engagement track (specific to Radion onboarding pilot)

- ~~`add-team-onboarding-canvas`~~ — *cancelled 2026-05-08, archived as [`archive/2026-05-08-add-team-onboarding-canvas/`](archive/2026-05-08-add-team-onboarding-canvas/). Engagement deprioritised against MVP v1 critical path; Phase 1 never started. Reusable artefacts harvested to [`_methodology/research/loop-protocol.md`](_methodology/research/loop-protocol.md) and [`_methodology/research/multi-author-canvas-instructions-template.md`](_methodology/research/multi-author-canvas-instructions-template.md).*

### Connections — credentials & UI

- `add-linear-oauth-connection` *(RAP-112, 2/6 phases done)* — per-user "Connect with Linear" on `/settings`. Bot path works; OAuth is the actor-bound layer for Telegram bridge.

### Network platform

- `add-network-public-surface` *(RAP-96, scaffold deferred)* — public-facing network presentation.
- `add-network-studio-surface` *(RAP-97, scaffold deferred)* — internal studio network views.

### Telegram canvas

- `add-tg-canvas-surface` *(RAP-121, Phase 4/6 in flight)* — Telegram canvas surface (replaces Web Speech voice input as the primary low-friction capture path). Phases 1 (G4 close, #358), 2 (DB migrations, #361), 3 (headless Muse runtime, #367), 4 (Telegram webhook + Whisper + bot identity, #370) shipped 2026-05-08. Phase 5 Studio UI surface + Phase 6 engagement opt-in + archive remain.

### Studio control room (admin)

- `add-studio-control-room` Phase 1 *(✅ shipped via #376, archived 2026-05-09 as `archive/2026-05-09-add-studio-control-room/`)* — owner-only `/control` + `/control/roadmap` surface. The Roadmap dashboard renders six sections from the live `openspec/ROADMAP.md` via the bundle generator's three new exports (`ROADMAP_TEXT` / `CHANGES_INDEX` / `ARCHIVE_COUNT`). Phases 2–5 = engagement / throughput / finance / team dashboards, each its own change folder, lit up by editing the single `control/page.tsx` file:
  - `add-control-engagement-dashboard` *(Phase 2, ✅ shipped via #389, archived 2026-05-09 as `archive/2026-05-09-add-control-engagement-dashboard/`)* — `/control/engagements` over `projects` + per-engagement methodology yaml + activity signals (canvas + forge). Two of five `<DashboardCard>` cards on `/control` are now `ready` (Roadmap + Engagements); three remain `planned`.
  - `add-control-throughput-dashboard` *(Phase 3, deferred)* — `/control/throughput` over PR/Linear closed-per-week + change-folder velocity.
  - `add-control-finance-dashboard` *(Phase 4, deferred)* — `/control/finance` over token spend per project + lead conversion.
  - `add-control-team-dashboard` *(Phase 5, deferred)* — `/control/team` over pending invitations + specialist network.

### Foundation (philosophy)

- `add-philosophy-foundation` *(RAP-148, Phase A scaffolded via #350)* — agents-overview + charter prose foundation. Phase B (manifesto) blocked on Pavel content.

---

## 4. Deferred / Future

Explicitly out of scope for MVP v1.x. Tracked here so they don't get lost.

### v1.2 / v1.3 milestones

- `add-muse-scout-mode` *(scaffold shipped 2026-05-05 — RAP-64; v1.3 design lock)* — codebase scanning → OpenSpec migration report. Implementation gated on RAP-48 final archive + ≥ 5 dogfood changes.

### Capture & Research enrichment (Canvas)

- `add-muse-competitor-research` *(not scaffolded)* — `find_competitors`, `analyze_competitor`, `competitive_landscape`. Renders comparison matrix as `canvas_files` row under `research/competitors.md`.
- `add-business-model-templates` *(not scaffolded)* — knowledge base of business model patterns (SaaS, marketplace, freemium, transactional, agency). Storage TBD.
- `add-canvas-multi-view` Wave 2+ *(deferred)* — Mirror viewer, file browser, spec graph stub.

### Canvas polish (deferred until preconditions clear)

- `add-canvas-view-switcher` *(deferred until second view exists)* — segmented control + localStorage persistence.
- `add-canvas-mirror-view` *(deferred until Mirror density grows)* — Mirror viewer placeholder.
- `add-canvas-voice-input` *(scaffold archived 2026-05-05 — RAP-51, superseded direction)* — Web Speech path; superseded by `add-tg-canvas-surface` for primary capture, may revive as desktop secondary.

---

## 5. Foundation — shipped (durable history)

Compact list. Each item is archived under `openspec/archive/<date>-<slug>/`.

- ✅ **RAP-48 — `add-forge-builder-mode`** — Forge as autonomous platform architect (10 commands; identity model; 5-checkpoint Builder execution; lessons-learned). All waves shipped; closed 2026-05-09. Archive: `openspec/archive/2026-05-05-add-forge-builder-mode/` + follow-up sync change `openspec/changes/sync-forge-builder-spec/`.

**DB foundation:**

- `add-projects-table` (RAP-43), `add-canvas-file-lifecycle` (RAP-44), `add-mirror-runtime-fix` (RAP-49), `reset-and-squash-database`, `add-canvas-and-session-split`.

**Bot identities:**

- `add-github-bot-and-creds` (RAP-47) — `rapoport-studio-bot` GitHub user, fine-grained PAT in Infisical, Linear bot member, `LINEAR_BOT_API_KEY`.

**Muse multi-tool:**

- `add-muse-conversation-discipline` (RAP-52), `add-muse-github-read-tools` (RAP-53), `add-muse-linear-read-tools` (RAP-54), `add-muse-github-write-tools` (RAP-60), `add-muse-linear-write-tools` (RAP-61), `add-muse-scout-mode` scaffold (RAP-64), `add-muse-mirror-authoring-tool`, `add-muse-research-tools-with-personas`, `add-muse-knowledge-system`, `wire-muse-runtime-in-studio`.

**Project bindings, home & adoption:**

- `add-project-bindings` (RAP-55+56), `add-project-home` (RAP-57), `add-project-home-mirror-methodology` (RAP-65), `fix-projects-rail-entry` (RAP-68), `add-adopt-existing-projects` (RAP-63), `add-project-connections` (RAP-134 — pluggable connection registry under `packages/connections`, GitHub + Linear kinds, ConnectionsList swap, creation-time GitHub on `<NewProjectForm>`; archived 2026-05-08).

**Design system + entity layer:**

- `add-entity-system` (RAP-91), `add-entity-card-primitive` (RAP-92 — `<EntityCard>` primitive + 38-entry registry + i18n placeholders; archive folder backfilled 2026-05-09), `migrate-surfaces-to-entity-card` (RAP-105 — 11 phases).

**Forge plumbing:**

- `add-forge-multi-project` (RAP-85), `add-forge-convention-adapter` (RAP-86), `add-forge-ui-task-detail` (RAP-130 Tracks A–D), `add-forge-discovery-surface` (Track C — F rail + `/tasks` index + project Forge runs card; archived 2026-05-08).
- `extend-rap63-backfill-unbind` (RAP-87) — `unbind` row + bindings + Mirror placeholder for Section 2.

**Canvas vertical:**

- `add-canvas-multi-view` (Wave 1), `add-canvas-state-machine` (RAP-133 Wave 2A), `add-canvas-stage-primitive` (RAP-80 Wave 1), `add-canvas-progress-view` (RAP-80 Wave 2), `add-canvas-voice-whisper` (cancelled / superseded by RAP-121), `add-canvas-idea-capture` (RAP-58 — sticky-note board, free-capture surface), `add-canvas-discovery-wizard` (MVP v1 Track B — six-field structured intake panel + Muse system-prompt preamble + `wizard_field_min_chars` lifecycle check; archived 2026-05-08), `add-canvas-invitations`, `add-canvas-diff-and-save`, `add-canvas-file-panel-server-actions`, `add-tool-status-streaming-ui`, `complete-canvas-file-lifecycle`.

**Network platform substrate:**

- `add-network-data-schema`, `add-network-channel-ingestion`, `add-network-entities`.

**Code display:**

- `add-studio-code-display` (RAP-127 + RAP-129 Phases 1–2) — Shiki client-lazy in `<Markdown>`.

**Founder & community surfaces:**

- `add-studio-self-serve-signup`, `expand-home-with-services-and-community` (RAP-82), `add-founder-track` (RAP-98).

**Operational dashboards:**

- `add-studio-control-room` Phase 1 — owner-only `/control` + `/control/roadmap` surface, X rail entry, `tools/build-openspec-bundle.mjs` extended with `ROADMAP_TEXT` + `CHANGES_INDEX` + `ARCHIVE_COUNT`, hand-rolled `parseRoadmap` + typed `RoadmapDriftError`, six-section dashboard with three sub-primitives (StatusPill / ProgressBar / FlowStep). Archived 2026-05-09.
- `add-control-engagement-dashboard` Phase 2 — `/control/engagements`, server-rendered. `loadEngagementOverview` joins `projects` + `METHODOLOGY_PROJECTS_BUNDLE` (per-engagement YAML, parsed with `js-yaml` + Date-coercion helper) + activity signals (`canvas_sessions` + `forge_runs`, in parallel). `<EngagementsDashboard>` + `<EngagementRow>` reuse Phase 1's locked status-token mapping (Decision D-7). 14 loader tests + 5 row-component tests. Two of five control-room cards now `ready`. Archived 2026-05-09.
- Phases 3–5 (`add-control-throughput-dashboard` / `add-control-finance-dashboard` / `add-control-team-dashboard`) deferred; each ships its own change folder.

**Methodology + miscellaneous:**

- `add-mirror-render-pipeline`, `add-mirror-schema-and-loader`, `add-methodology-templates`, `add-client-onboarding`, `add-client-role`, `add-pending-invitations-surface`, `credit-token-cost-to-canvas-budget`, `add-user-settings`, `enrich-status-taxonomy`, `enrich-tactical-grammar`, `enrich-surface-density`, `reconcile-token-names`, `replace-listusers-with-rpc`, `sync-design-system-with-reality`, `sync-design-system-post-25-26`, `mvp-v1-architect-and-studio-narrow`, `mvp-v1-design-system-studio-variant`, `mvp-v1-home-blocks-completion`, `refactor-forge-consumer-registry`, `refactor-forge-persistence-adapter`, `spec-drift-cleanup-foundation`, `add-project-migration-prompt` (Branch-B template at `_methodology/projects/_template-migration-prompt.md` — generic foreign-dialect → studio-prose migration prompt; per-engagement forks for Vivod / Dentour fork from this; archived 2026-05-09).

---

## 6. Drift / cleanup items

Surfaced during 2026-05-08 ROADMAP sync. Each is small enough to fold into the next-touching change rather than a dedicated PR.

1. ~~**RAP-92 missing archive folder**~~ — *resolved 2026-05-09 via retroactive backfill: [`archive/2026-05-06-add-entity-card-primitive/`](archive/2026-05-06-add-entity-card-primitive/) reconstructs proposal/design/tasks from PR #234's body. RAP-105 cross-reference no longer dangles.*
2. **GitHubClient consolidation drift (resolved)** — canonical `GitHubClient` now at `packages/forge/src/core/github-client.ts` after RAP-140 atomic swap; `apps/studio/src/lib/github-client.ts` and `apps/studio/src/lib/linear-client.ts` deleted (PR #336).
3. **`db.md` historical drift items 1–5** — most resolved by `reset-and-squash-database` (2026-05-05). Re-verify on next pre-flight.
4. **Memory snapshot Supabase project ID and Linear team ID** — corrected via `memory_user_edits` post-merge.

---

## Quick reference — current state at a glance

```
═════════════════════════════════════════════════════════════════════════════
  MVP v1 (Section 1)         🟡 ~75%  Tracks B + C ✅; A 80%; D 0%
                                       ETA 1–2 days
  MVP v1.1 — Unbind dogfood  🟡 ~57%  4/7 children of RAP-83 done
  Parallel tracks (§ 3)      🟡 mix   tg-canvas Phase 6, xstate Wave 2B
  Foundation (§ 5)           ✅ 76    archived change folders
  Active changes             🟡 11    in openspec/changes/ on this branch
  Total scope ever opened    ~87      76 archived + 11 active (post-2026-05-09 engagement-dashboard archive on top of migration-prompt + control-room + RAP-92 backfill)
═════════════════════════════════════════════════════════════════════════════
```

To see the live picture, run `ls openspec/archive/ | wc -l` and `ls openspec/changes/ | wc -l` against the repo. **For "what shipped today", run `gh pr list --state merged --search "merged:>=YYYY-MM-DD"` — `ROADMAP.md` is a planning artefact, not a status artefact, and lags behind the merge stream by hours.**
