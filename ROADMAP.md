# Roadmap — Studio platform v1 → v2

This roadmap is the master sequencing document. It is organised around **MVP milestones**, not feature areas: the top sections are what blocks the next reviewable cut of the platform; the bottom sections are durable history and deferred work.

**Last full sync:** 2026-05-12 morning · OpenSpec ↔ Linear coverage gap-closure pass. **15 new RAP issues filed (RAP-549..RAP-563)** to close the 14-folder Linear-coverage gap surfaced by the audit (active OpenSpec changes that previously carried `RAP-???` / `RAP-XXX` / `Linear issue: TBD` / `Owner: Pavel (TBD)` placeholders, plus `redesign-canvas-progress-component` whose original RAP-369 had already shipped to Done). Each new ticket is shaped exactly the way `forge inspect` parses (slug reference, `## Relevant specs` block, `Depends on …` prereqs, `auto-build` label, Pavel as assignee). 7 `blockedBy` edges wired in Linear so `forge epic --close` can topo-sort the vocabulary epic (RAP-549 → 550/551/552) and the home-block IA cascade (RAP-559 → 558 → 560 → 561). Bonus: two missing `design.md` files added (`forge-w2-followups`, `update-forge-inbox-default-state`) for prose-convention compliance. Linear coverage of active OpenSpec moved from **48% → 96%** (26/27 active changes now filed; the remaining 1 — `define-research-domain-entity` — is intentionally idea-stage per `openspec/AGENTS.md`). Tooling captured under `tools/` for the next pass: `audit-openspec-linear-coverage.mjs`, `file-linear-coverage-issues.mjs`, `link-linear-issue-deps.mjs`, `update-openspec-rap-refs.mjs`. **Previous full sync (2026-05-10 evening):** post-#634 (~30 PRs since 2026-05-11 morning sync). **RAP-229 redesign-intake-form closed end-to-end** — promote `IntakeForm` from `apps/web/src/components/contact/` to `@rapoport-studio/ui` with EN/RU/RO i18n + WCAG 2.2 AA via axe-core. Phase 1 RAP-230 RadioCardGroup (#562), Phase 2 RAP-231 ChipRadioGroup (#565), Phase 3a RAP-232 scaffold (#570), Phase 3b RAP-235 blocks (#608), Phase 3c+7 RAP-237 orchestrator + draft persistence + tests (#612), Phase 4a RAP-233 EN authoritative (#590), Phase 4b RAP-236 RU+RO + parity (#598), Phase 5 RAP-238 rewrite `/contact` + `/discovery` (#615 — net −354 LOC, deletes legacy 292-LOC form), Phase 6 RAP-234 coded error contract (#591), Phase 8+9 manual RAP-239 (#618/#619/#620 archive + `web.md`/`i18n.md` amendments). **RAP-268 add-url-state-overlay-system closed end-to-end** — S1 a11y baseline RAP-283..293 (skip-link, DirectionProvider, axe-Playwright fixture, ESLint rules, PointerEventsWatchdog, SonnerWrapper, view-transitions utility, LiveRegions+announce, types) shipped #581/#582/#583/#584/#585/#586/#587, S2 overlay primitives RAP-294..298 + RAP-300 useDialog (#592) + RAP-301 announce subpath (#609), S3 capstone RAP-302 OverlayHost (#602), S4 mounts RAP-303 studio (#605) + RAP-304 web (#607), S5 reference module RAP-305 invite-member.dialog (#610), S6 specs RAP-306 `linkable-ui.md` (#617) + RAP-307 `ui.md`/`design-components.md` (#621) + RAP-308 `platform.md`/`web.md`/`studio.md` (#620), archive RAP-309 (#624). **Home epic RAP-255 partial** — RAP-263 OperatingModel block (#603 + a11y fix #625), RAP-265 ForFounders geo-tags + diaspora hook (#604). **Production-deploy unblock cascade** — RAP-238 + RAP-302 surfaced four runtime gaps that the unit gates missed: NuqsAdapter mounts (#611), comprehensive Suspense + next-intl + Server/Client wrapper (#623, supersedes earlier narrow #622), authenticated-SSR overlay-host wrapper (#626), `intake` i18n namespace registration (#628), OPERATING_MODEL_STATS_KV provisioning (#630). **New scaffolds** (all 0%–30% on tasks.md, gated on owner-pickup) — `add-cli-standards`, `add-frontend-runtime-standards` (#634 adoption section/RAP-280), `add-media-block-primitive` (#596), `add-research-authorship-convention` (#627), `define-chart-system` (#629/RAP-342), `define-research-domain-entity` (#631), `extend-philosophy-foundation-trilingual`, `redesign-button-system-orange-primary`, `redesign-canvas-progress-component` (#633/RAP-369). **RAP-410 redesign-tasks-surface closed end-to-end** (2026-05-11) — six-wave Forge sprint promoting `/tasks` from list-router to Linear-grade working list. W0 spec corrections RAP-411 (#684), W1 batched Linear loader + `LinearClient.fetchIssuesBatch` RAP-412 (#688), W2 `<TaskListRow>` + `<MiniPhasePipeline>` RAP-413 (#696 — salvaged manually from a Forge build that stalled at the push checkpoint pre-RAP-420 fix), W3 inline row expansion + `?expanded=` URL state RAP-414 (#700), W5 `/tasks/[key]` parity + a11y + extracted `<PropertyCell>` primitive RAP-416 (#702), W4 filter strip + auto-clear-on-expand banner RAP-415 (#703). Archive: `openspec/archive/2026-05-11-redesign-tasks-surface/`; amendments added to `openspec/current/studio-workspace/README.md § "Task surface v2"` and `openspec/current/forge/entities.md § Linear API` (fetchIssuesBatch). **Forge fleet enhancements** — RAP-269 STALE_REFERENCE/OPERATIONAL/HOT tier constants (#632). Archive: **112 folders**. Active: **14** (4 gated engagements `add-project-connect-onebutton`/`migrate-unbind-into-studio`/`onboard-vivod`/`onboard-dentour` + 10 scaffold-only).

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

The Spec phase is mature (Architect mode, multi-tool Muse, full convention adapter). Capture is complete (sticky-note idea board shipped via RAP-58; structured discovery wizard shipped end-to-end via `add-canvas-discovery-wizard`, archived 2026-05-08). Research is partial (competitor research not yet scaffolded). Build phase has the runtime substrate (Forge UI surface RAP-130 + `add-forge-discovery-surface` Track-C close-out, multi-project config RAP-85, convention adapter RAP-86); the executor (RAP-48 Builder) shipped end-to-end (Waves 1–3 + Wave 4 Phases A/B/D, RAP-48 closed 2026-05-09). **Track D Cycle 1** (first real-issue dogfood, RAP-167) and **`sync-forge-builder-spec` Wave 4** are archived 2026-05-09; **RAP-172** lit the Implement body on `/tasks/[key]` (`archive/2026-05-09-add-forge-builder-orchestrator-ui/`).

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
| 4 | Forge UI — Plan / Review / PR | ✅ 90% | `/tasks/[key]`, DB, 15 actions, SSE, Phases 1/3/4 + Track-C discovery surface (F rail + `/tasks` index + project Forge card, all archived 2026-05-08); Forge engine arc closed via RAP-213 Phases 2-6 (model registry / per-role budgets / retry-on-gate / AI reviewer / archive) 2026-05-10 | Polish |
| 5 | Forge UI — Implement (Phase 2) | ✅ 100% | `<LiveBuilderPanel>` + studio `PersistenceAdapter` + SSE snapshot extension (`archive/2026-05-09-add-forge-builder-orchestrator-ui/`, RAP-172) | In-UI start/cancel deferred |
| 6 | Forge Builder Mode (CLI) | ✅ 100% | Waves 1–3 ✓ (PRs #161/#166/#186) + Wave 4 Phase A ✓ (#351) + Phase B ✓ (RAP-151 sandbox seed 2026-05-08) + Phase 3.A drift fixes via #378/#379/#382/#385/#387/#391/#394 — RAP-48 closed 2026-05-09. **`/forge` slash command shipped 2026-05-10 (#531)** — parallel build dispatch with `~/.forge/state/<KEY>.json` fleet dashboard. | — |
| 7 | Studio dogfood | 🟡 in-flight | Cycle 1 ✅ archived (`archive/2026-05-09-sync-roadmap-post-rap48-close/`, RAP-167); `/projects/studio` substrate ✅ archived (`archive/2026-05-09-migrate-studio-into-platform/`, RAP-169 Wave 6); **2026-05-10 Forge dispatched 7+ RAP issues end-to-end via `/forge`** (RAP-96/97/188/190/216/217/etc. — most as Builder runs in detached background, several as no-op confirmations on already-shipped work). Full **UI-first** close-the-loop dogfood still open. | Cycles 2 + 3 + retro + one RAP closed entirely via studio UI |

### Critical-path tracks

```
Track A — RAP-48 close-out via      All waves merged; RAP-48 closed         [████████████████████] ✅ shipped
          sync-forge-builder-spec   2026-05-09; Wave 4 archived (incl.
                                    Phase C = RAP-167 dogfood)
Track B — add-canvas-discovery-     All phases shipped + archived          [████████████████████] ✅ shipped
          wizard                                                              (archived 2026-05-08)
Track C — add-forge-discovery-      F-rail + /tasks index + project card   [████████████████████] ✅ shipped
          surface                                                             (archived 2026-05-08)
Track D — Studio dogfood E2E        Cycle 1 ✅ RAP-167 archived.           [█████████░░░░░░░░░░░] Cycles 2–3
          (= sync-forge-builder-    Cycle 2 substrate ✅ (RAP-169 data
          spec Phase 3)             registration). Cycle 2 UI-first proof
                                    + Cycle 3 + MVP proof remain.
```

#### Track A — `add-forge-builder-mode` (RAP-48, ✅ closed 2026-05-09)

Forge as platform architect: own GitHub/Linear/Supabase identities, six commands (`inbox` / `inspect` / `plan` / `build` shipped Waves 1–3; `status` / `cancel` shipped Wave 4 Phase A (#351); **RAP-48 closed 2026-05-09**), Anthropic Agent SDK in working tree.

**Hard prerequisites — all met as of 2026-05-08:**

- ✅ RAP-47 — bot identities
- ✅ Canonical `GitHubClient` consolidated in `packages/forge/src/core/github-client.ts` (RAP-136 / RAP-140)
- ✅ `add-project-connections` shipped end-to-end (archived 2026-05-08 as `archive/2026-05-08-add-project-connections`; operational tail — production deploy, manual smoke, follow-up `drop-deprecated-projects-binding-columns`, RAP-111 close — tracked separately)

**Already shipped (PRs #161 / #166 / #186, 2026-05-04 → 2026-05-05):**

- Wave 1 — identity foundation (`loadForgeIdentity` → `core/auth.ts`), `forge inbox`, `forge inspect`, programmatic API on `Forge` handle.
- Wave 2 — `forge plan` + lessons-learned mechanism (`~/.forge/lessons-learned.md`) + state model (`~/.forge/state/*.json`) + Builder prompt template (`packages/muse/src/builder-prompt.ts`).
- Wave 3 Phase 8 — build orchestrator + Anthropic Agent SDK call shape (per spike `archive/2026-05-05-add-forge-builder-mode/spikes/2026-05-05-claude-agent-sdk.md`) + six-checkpoint gating.

**Wave 4 — fully shipped, archived `archive/2026-05-09-sync-forge-builder-spec/`:**

- ✅ Phase A — `forge status` + `forge cancel` orchestrators + in-process `build-registry.ts` + `GitHubClient.deleteBranch` (merged via #351 on 2026-05-08).
- ✅ Phase B — sandbox lessons-seed dogfood completed via RAP-151 (2026-05-08); seed entry written to `~/.forge/lessons-learned.md`.
- ✅ Phase C — first real-issue e2e (Track D Cycle 1) completed via RAP-167; archived `archive/2026-05-09-sync-roadmap-post-rap48-close/` (manual fallback PR path documented there).
- ✅ Phase D — Phase 3.A drift fixes shipped via #378/#379/#382/#385/#387/#391/#394 between 2026-05-08 and 2026-05-09; RAP-48 closed 2026-05-09.

Note: `sync-forge-builder-spec` lived under `openspec/changes/` while in flight; it is now archived. RAP-48 was prematurely closed 2026-05-05 after Wave 3 Phase 8 merged; reopened to "In Progress" 2026-05-08 to absorb Wave 4 + truthing.

Estimated remaining effort: **complete** for RAP-48 and Wave 4 Phases A–D. Remaining MVP v1 gap is **Step 7** — additional dogfood cycles + proven UI-first loop (see §1 table).

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

#### Track D — Studio dogfood E2E

**Cycle 1** ✅ archived 2026-05-09 (`archive/2026-05-09-sync-roadmap-post-rap48-close/`, RAP-167) — first real-issue end-to-end run via the CLI loop, manual fallback PR path documented in the archive.

**Cycle 2** — split into two halves:

- **Cycle 2 substrate** ✅ archived 2026-05-09 (`archive/2026-05-09-migrate-studio-into-platform/`, RAP-169). Studio is now a registered `projects` row with `project_connections` (GitHub + Linear), engagement profile (`_methodology/projects/studio.yaml`), and Mirror placeholder (`mirror/studio/perception.md`). `/projects/studio` resolves with the Connections card lit. Wave 5 (UI-first dogfood) deferred to its own session and tracked below.
- **Cycle 2 UI-first dogfood** — open. Pick any small RAP-* issue (a doc fix, a one-file polish, a typo). Drive the full loop **inside `app.rapoport.studio`** rather than `pnpm forge`:

  ```
  /canvases → create canvas → assign project=studio
  Architect mode → emit proposal/design/tasks
  [Convert to project] → branch on rapoport-studio/rapoport.studio
  /tasks/<RAP-XXX> → Plan / Implement / Review
  → merge PR → archive change folder
  → append observations to openspec/_research/studio-self-hosting-first-run.md
  ```

  Verifiable surfaces: `<ForgeRunsCard>` on `/projects/studio` shows the run; the canvas appears in "Recent canvases"; PR opened against `main`. Estimated effort: **2–4 hours** + drift fixes for whatever surfaces flicker. **Definition of done for MVP v1.**

**Cycle 3** — second UI-first dogfood pass, ideally on a slightly larger RAP-* issue (multi-file change). Surfaces drift the first dogfood didn't trigger; closes the loop on "the studio runs itself through the studio UI" as a stable claim.

### Dependency graph

```
Track C (forge-discovery-surface)        ✅ done & archived 2026-05-08
Track B (canvas-discovery-wizard)        ✅ done & archived 2026-05-08
                                                      │
                                                      ▼

Track A (sync-forge-builder-spec)      ✅ Wave 4 + archive 2026-05-09
   │
   ▼
Track D (Studio dogfood E2E)             Cycle 1 ✅ (RAP-167) — Cycle 2 substrate ✅ (RAP-169) — Cycle 2 UI-first + Cycle 3 open
                                                      │
                                                      ▼
                                              🎯 MVP v1 — Step 7 + UI-first proof
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

- ~~`add-linear-oauth-connection`~~ *(RAP-112 ✅ closed 2026-05-10, archived as `archive/2026-05-10-add-linear-oauth-connection/`)* — all seven phases shipped end-to-end. Schema + AES-256-GCM crypto helper (#353), OAuth routes (#452), `<IntegrationsCard>` UI on `/settings`, actor-bound `loadLinearForUser()` helper + Telegram-bridge precedence (#486), spec amendments + archive (#545).

### Network platform

- ~~`add-network-public-surface`~~ *(RAP-96 ✅ closed 2026-05-10, archived as `archive/2026-05-10-add-network-public-surface/`)* — Wave 3 close-out: live Supabase fetchers, dagre LR mind-map, click-to-detail Sheet + URL deep-link, mobile tier-grouped fallback, locales en+ru parity, all sections rendering EntityCard variants. Sections shipped via #529 (PR A) + #538 (Wave 3 close-out); archive #546. Playwright e2e carved out to follow-up.
- ~~`add-network-studio-surface`~~ *(RAP-97 ✅ closed 2026-05-10, archived as `archive/2026-05-10-add-network-studio-surface/`)* — Phase 4 close-out: `skill_specialist_counts` materialized view + daily `pg_cron` refresh, four server-action bodies, 17 unit + 5 RTL tests, `studio.md § Network surface` amendment. Shipped via #537; archive #544. Playwright e2e carved out to follow-up.

### Telegram canvas

- `add-tg-canvas-surface` *(RAP-121 ✅ archived 2026-05-09 as `archive/2026-05-09-add-tg-canvas-surface/`)* — per-canvas TG Muse surface (`@rs_canvas_bot`), headless runtime, propose-only mutations, daily cap, engagement opt-in. Shipped via #358 / #361 / #367 / #370 / #373 + spec amendments.

### Studio control room (admin)

- `add-studio-control-room` Phase 1 *(✅ shipped via #376, archived 2026-05-09 as `archive/2026-05-09-add-studio-control-room/`)* — owner-only `/control` + `/control/roadmap` surface. The Roadmap dashboard renders six sections from the live `openspec/ROADMAP.md` via the bundle generator's three new exports (`ROADMAP_TEXT` / `CHANGES_INDEX` / `ARCHIVE_COUNT`). Phases 2–5 = engagement / throughput / finance / team dashboards, each its own change folder, lit up by editing the single `control/page.tsx` file:
  - `add-control-engagement-dashboard` *(Phase 2, ✅ shipped via #389, archived 2026-05-09 as `archive/2026-05-09-add-control-engagement-dashboard/`)* — `/control/engagements` over `projects` + per-engagement methodology yaml + activity signals (canvas + forge). Two of five `<DashboardCard>` cards on `/control` are now `ready` (Roadmap + Engagements); three remain `planned`.
  - `add-control-throughput-dashboard` *(Phase 3, deferred)* — `/control/throughput` over PR/Linear closed-per-week + change-folder velocity.
  - `add-control-finance-dashboard` *(Phase 4, deferred)* — `/control/finance` over token spend per project + lead conversion.
  - `add-control-team-dashboard` *(Phase 5, deferred)* — `/control/team` over pending invitations + specialist network.

### Foundation (philosophy)

- ~~`add-philosophy-foundation`~~ *(RAP-148 Phase A ✅ closed 2026-05-10, archived as `archive/2026-05-10-add-philosophy-foundation/`)* — agents-overview + charter prose foundation shipped via #350 (2026-05-08); archive #543. Phase B (Pavel-voice manifesto) carved out — when content is ready, author a new change folder (e.g., `add-philosophy-manifesto`) rather than re-opening the archive.
- ~~`extend-philosophy-foundation-trilingual`~~ *(RAP-179/RAP-195/RAP-196 ✅ archived 2026-05-13 as `archive/2026-05-13-extend-philosophy-foundation-trilingual/`)* — Phases 1 (spec), 3 (EN), 4 (RO) complete. Phase 2 (RU canonical body) operator-blocked; `manifesto.ru.md` must be authored by Pavel in a follow-up PR on `feat/rap-179/manifesto-ru`.

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
- Forge builder hardening (all 2026-05-09): RAP-162 Anthropic streaming + delta (#451), RAP-163 auto-PR-body generation (#453), RAP-159 vitest coverage gate 60% (#454), RAP-168 worktree deps install before Builder launch (#455), RAP-150 Maestro Phase 1.1 `allowedToolNames` per mode (#445).
- **Forge engine RAP-213 (`improve-forge-model-tiering-and-verifier`)** — closed end-to-end 2026-05-10, archive `2026-05-10-improve-forge-model-tiering-and-verifier/`. Phase 2 model registry refresh + per-role config (#519, RAP-214). Phase 3 per-role cost guards + `costByRole` tracking (#524, RAP-215). Phase 4 retry-on-gate-failure with exponential backoff (#530, RAP-216). Phase 5 AI reviewer (Haiku→Sonnet escalation) + spec-drift metrics (#533, RAP-217). Phase 6 archive (#534).
- `add-forge-worktree-pnpm-install` (RAP-168 Phase B) — `installWorktreeDeps()` between `setupWorktree()` and Builder launch + lockfile-hash skip optimisation; archived 2026-05-10 (#527).
- **Forge dispatch tooling (2026-05-10)** — `/forge` slash command for parallel build orchestration (#531), global build lock at `~/.forge/locks/build.lock` (#522, RAP-224), cancel terminal-state guard (#515 + #517, RAP-218), gate-eval rejects compound shell commands (#494, RAP-199), bundle auto-regen in build post-step (#493, RAP-198).

**Maestro substrate (2026-05-10):**

- DB-backed routing journal (#536, RAP-187, archive `2026-05-10-add-maestro-db-journal-fixme-renamed/`); agent status table (#489 + archive #491-area, RAP-186, archive `2026-05-10-add-agent-status-table/`); orchestra journal table (#491, RAP-185, archive `2026-05-10-add-orchestra-journal-table/`); orchestra storage scaffolds (#483, RAP-184, archive `2026-05-10-add-orchestra-storage-scaffolds/`); Maestro canvas mode (#495, RAP-189, archive `2026-05-10-add-maestro-canvas-mode/`); routing plan card (#539, RAP-190); pure `route()` post-processor (#540, RAP-188, archive `2026-05-10-add-maestro-route-postprocessor/` via #542); audit populator (#502, RAP-202); journal replay CLI (#501, RAP-201); routing-plan parser fixtures (#503, RAP-200). Maestro substrate archives consolidated via #497 + #496 (`define-design-system-spec-convention` archive 2026-05-10).

**Design system token coherence umbrella (2026-05-10):**

- `improve-design-system-token-coherence` — umbrella draft (#498), archive 2026-05-10 (#508, RAP-209). G1 radius + grid + accent rename (#507, RAP-206 — D1+D2+D4). G2 package-coverage tokens (ink-light / blueprint-light / code-bg) split across #504 + #505 (RAP-207 — D6+D7+D8). G3 ink-muted alignment + apps/web hex→oklch (#514, RAP-208 — D3+D5).
- Motion tokens registry (#500, RAP-204 — D3) added to `design-tokens.md`.
- Layout primitives — `Container` / `Stack` / `Grid` / `Spacer` (#506, RAP-211 — F6).
- Shadcn primitives restored — `Checkbox` / `Switch` / `Spinner` / `Alert` (#513, RAP-205 — F8) + `components.json` for `npx shadcn add` (#510, RAP-212 — F1).
- Legacy brand split — moved to `@rapoport-studio/legacy-brand` (#512, RAP-210).
- Chip family unified — single `<Chip>` primitive with kind taxonomy (#523, RAP-223 — F1).
- PostCard migrated to `<EntityCard variant="card">` (#520, RAP-221 — F2.2).
- Track-aware home blocks — `track` prop on Hero / HowWeWork / ClosingCTA / ServicesSection (#516, RAP-219 — F4).
- Component-reuse audit closed 8/8 (#525).

**Studio chrome consolidation (2026-05-10):**

- Labelled sidebar (#482, RAP-194) + archive #484. Sidebar + shared brand mark merged (#488, RAP-197) + archive #492. `<SectionHead>` promoted to `@rapoport-studio/ui` (#485, RAP-203) + archive #487.
- Hotfixes — Tailwind 4 max-w-* scale (canvas chat + Invite dialog sliver, #532, RAP-225) + canvas chat + modal width regression (#526) + how-we-work + closing-cta typecheck (#521).

**Studio as project:**

- `migrate-studio-into-platform` (RAP-169) — studio onboarded as `/projects/studio` (Wave 6 close-out: `projects` row + `project_connections` + Mirror placeholder + engagement profile); archived 2026-05-09 as `archive/2026-05-09-migrate-studio-into-platform/`. Wave 5 UI-first dogfood proof deferred to Track D Cycle 2.

**Linear OAuth (RAP-112, closed 2026-05-10):**

- All seven phases shipped end-to-end. Phase 3 schema + AES-256-GCM crypto helper (#353, RAP-114). Phase 4 OAuth flow routes — connect / callback / disconnect (#452, RAP-115). Phase 5 `<IntegrationsCard>` UI on `/settings`. Phase 6 actor-bound helper `loadLinearForUser()` + Telegram-bridge precedence — user OAuth preferred over bot when row exists (#486, RAP-117). Phase 7 archive + spec amendments to `db.md` / `studio.md` / `platform.md` / `auth.md` (#545). Archive `2026-05-10-add-linear-oauth-connection/`.

**Network surfaces (closed 2026-05-10):**

- `add-network-public-surface` (RAP-96) — Wave 3 closed: live Supabase fetchers, dagre LR mind-map with click-to-detail Sheet + URL deep-link `?node=<kind>:<id>` + `< 768px` tier-grouped fallback, locales en+ru parity (ro placeholder), nine sections rendering EntityCard variants. Phase 3 PR A (#529 sections 2.1/2.5/2.6/2.7/2.9/2.10) + Phase 3 PR B (#538 Wave 3 close-out) + archive #546. Playwright e2e carved out to follow-up.
- `add-network-studio-surface` (RAP-97) — Phase 4 close-out: `skill_specialist_counts` materialized view migration with daily `pg_cron` refresh, four server-action bodies + 17 Zod-locked tests + 5 RTL section-render tests, `studio.md` § Network surface amendment (#537). Archive #544.

**Drift cleanup (closed 2026-05-10):**

- `add-philosophy-foundation` Phase A archived (#543). Agents-overview + charter amendment shipped via #350 (2026-05-08). Phase B (Pavel-voice manifesto) carved out to its own future change folder when content is ready.
- `fix-magic-link-cross-device` archived (#535) — work shipped via #425 + #440; spec amendments to `auth.md` for token-hash flow + OAuth (PKCE) flow split.
- `drop-deprecated-projects-binding-columns` archived (#528) — RAP-175, work shipped via #478.
- `add-maestro-route-postprocessor` archived (#542) — RAP-188 work shipped via #540.

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

Surfaced during ROADMAP syncs. Each is small enough to fold into the next-touching change rather than a dedicated PR.

1. ~~**RAP-92 missing archive folder**~~ — *resolved 2026-05-09 via retroactive backfill: [`archive/2026-05-06-add-entity-card-primitive/`](archive/2026-05-06-add-entity-card-primitive/) reconstructs proposal/design/tasks from PR #234's body. RAP-105 cross-reference no longer dangles.*
2. **GitHubClient consolidation drift (resolved)** — canonical `GitHubClient` now at `packages/forge/src/core/github-client.ts` after RAP-140 atomic swap; `apps/studio/src/lib/github-client.ts` and `apps/studio/src/lib/linear-client.ts` deleted (PR #336).
3. **`db.md` historical drift items 1–5** — most resolved by `reset-and-squash-database` (2026-05-05). Re-verify on next pre-flight.
4. **Memory snapshot Supabase project ID and Linear team ID** — corrected via `memory_user_edits` post-merge.
5. **Forge verifier — baseline-blind (RAP-227, surfaced 2026-05-10)** — `parseVerificationResults` treats any failing test as a regression introduced by the current PR. Observed false-positive on RAP-97 #537: 1/952 pre-existing test failure unrelated to RAP-97 caused `state.json: failed`. Fix queued as RAP-227: capture `~/.forge/builds/<KEY>/baseline.json` against `origin/main` HEAD, diff post-build, only fail on strict-superset.
6. **Forge anthropic.ts temperature regression (memory `forge_temperature_opus_47_regression.md`, surfaced 2026-05-10)** — `temperature: 0` hardcoded in `packages/forge/src/core/anthropic.ts:214` rejected by `claude-opus-4-7`. Workaround: `pnpm forge plan --sonnet`. Real fix is per-role temperature dispatch (omit for Opus 4.7, keep `0` for Sonnet builder + Haiku reviewer determinism). Linear issue not filed (Linear free-tier limit 2026-05-10); fold into next `packages/forge` PR.
7. **Forge re-dispatch on already-shipped commits** — observed 2026-05-10 on RAP-96 redispatch: original commit `9619d843f` was squash-merged to main as `64eaabb22` (different SHA), so the worktree branch still pointed at `9619d843f`, push opened duplicate PR #541 (closed). Forge's `--auto-push` should detect "branch HEAD content matches a recently-merged squash commit" and skip push. No Linear issue yet.
8. **Phase B manifesto for `add-philosophy-foundation`** — Phase A folder archived 2026-05-10 (#543) with Phase B narrative carved out. When Pavel-voice manifesto content is ready, author a new change folder (e.g., `add-philosophy-manifesto`) — do not re-open the archived folder (per memory `feedback_archive_frozen.md`).
9. **Network e2e carved out** — `add-network-public-surface` archive (#546) and `add-network-studio-surface` archive (#544) both deferred Playwright e2e to a wider `apps/web` / `apps/studio` e2e push. No separate change folder today.

---

## Quick reference — current state at a glance

```
═════════════════════════════════════════════════════════════════════════════
  MVP v1 (Section 1)         🟡 ~93%  Tracks A–C ✅; Wave 4 ✅; Implement UI ✅
                                       /projects/studio substrate ✅ (RAP-169)
                                       Forge engine RAP-213 closed end-to-end
                                       Step 7: Cycle 1 ✅ — Cycles 2–3 + UI loop
  MVP v1.1 — Unbind dogfood  🟡 ~57%  4/7 children of RAP-83 done
  Parallel tracks (§ 3)      ✅ done  TG canvas ✅; Linear OAuth ✅; network public ✅;
                                       network studio ✅; xstate Wave 2B remaining
  Foundation (§ 5)           ✅ 124   archived folders (`ls openspec/archive/ | wc -l`)
  Active changes             🟡 27    26 with filed Linear (RAP-549..563 batch)
                                       + 1 idea-stage (define-research-domain-entity)
  Linear coverage (active)   ✅ 96%   26/27 — `tools/audit-openspec-linear-coverage.mjs`
  Total scope ever opened    ~151     124 archived + 27 active
═════════════════════════════════════════════════════════════════════════════
```

To see the live picture, run `ls openspec/archive/ | wc -l` and `ls openspec/changes/ | wc -l` against the repo. **For "what shipped today", run `gh pr list --state merged --search "merged:>=YYYY-MM-DD"` — `ROADMAP.md` is a planning artefact, not a status artefact, and lags behind the merge stream by hours.**
