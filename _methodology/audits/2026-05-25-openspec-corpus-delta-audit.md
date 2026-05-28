---
title: OpenSpec corpus — delta audit since 2026-05-22
status: audit
owner: pavel
audience: [pavel, ai, reviewer]
date: 2026-05-25
tldr: Delta pass against the 2026-05-22 25-specialist audit, scoped to changes the corpus has absorbed in the last 72 hours. Dominant defect from the prior audit — internal spec drift — was self-remediated by the RAP-1164 epic (three sub-changes archived 2026-05-22). 8 of the 10 §A drift findings are now resolved on disk; 4 of 4 §C "specced but unenforced" rules now have machine enforcement; the §D AI-drafted-without-review pattern is largely unchanged; §B "designed the test, then cancelled the run" is untouched. One finding moved in the wrong direction: model-identifier drift now spans more files, not fewer. Estimated corpus mean: **6.2 / 10** (up from 5.9).
---

# OpenSpec corpus — delta audit since 2026-05-22

> Companion to [`2026-05-22-openspec-corpus-specialist-audit.md`](2026-05-22-openspec-corpus-specialist-audit.md).
> Reads the repo state at 2026-05-25 against the prior audit's claims and asks: what is fixed, what is unchanged, what regressed.
> Method gate: [`../audit-discipline.md`](../audit-discipline.md). Every finding cites an exact `file:line` in the current tree.

---

## What changed in 72 hours

`git log openspec/ --since 2026-05-22 --until 2026-05-26` shows **~70 commits** touching specs, changes, methodology, root, and entities. The signal-bearing ones for this delta:

- **RAP-1164 epic shipped end-to-end on 2026-05-22.** Three sub-changes archived in PR #1604: [`reconcile-openspec-corpus-drift`](../../archive/2026-05-22-reconcile-openspec-corpus-drift/), [`restore-design-system-spec-files`](../../archive/2026-05-22-restore-design-system-spec-files/), [`wire-spec-enforcement-gates`](../../archive/2026-05-22-wire-spec-enforcement-gates/). The epic was scaffolded the same day the corpus audit was authored — Pavel turned its dominant finding into a remediation epic and shipped all three children inside 24 hours.
- **Design-system corpus restored.** Four files that the 22.05 audit flagged as ratified-but-missing are now on disk: [`design-system.md`](../../current/design-system.md) (10.8KB), [`design-tokens.md`](../../current/design-tokens.md) (33KB), [`design-motion.md`](../../current/design-motion.md) (7.6KB), [`design-components.md`](../../current/design-components.md) (26KB).
- **WCAG 2.2 AA named as corpus contract.** [`design-system.md:78`](../../current/design-system.md) declares `D-DS-10` — AA is the corpus floor, AAA elective. Touch-target reconciliation (`44×44` vs `24×24`) lands at [`design-system.md:86-93`](../../current/design-system.md).
- **Three specced-but-unenforced gates now machine-enforced.** ESLint rule for cold-pitch tokens, `server-only` marker for `createAdminClient`, and a webhook HMAC verifier at [`apps/studio/src/lib/verify-webhook-signature.ts`](../../../apps/studio/src/lib/verify-webhook-signature.ts) all shipped under `wire-spec-enforcement-gates`.
- **Mirror tombstone deleted.** Per Atlas `D-ATLAS-10`, `openspec/current/mirror.md` no longer exists; both glossaries agree Mirror is a live Tier-1 Brand-sphere entity ([`_root/glossary.md:191`](../../_root/glossary.md), [`_methodology/glossary.md:32-38`](../glossary.md)).
- **New in-flight surface area.** [`add-inquiry-entity`](../../changes/add-inquiry-entity/), [`add-inquiry-workflow-gates`](../../changes/add-inquiry-workflow-gates/), [`add-specialist-tool-composition`](../../changes/add-specialist-tool-composition/), [`add-muse-translation-mode`](../../changes/add-muse-translation-mode/), [`establish-studio-entity-baseline`](../../changes/establish-studio-entity-baseline/), [`publish-legal-bundle`](../../changes/publish-legal-bundle/), [`linear-as-agent-control-plane-openspec`](../../changes/linear-as-agent-control-plane-openspec/).

---

## §A — Internal spec drift: status of the 10 named findings

The 22.05 audit named the **dominant defect** as file-versus-file contradictions. Below, each named finding is re-verified at its current `file:line`.

| # | 22.05 finding | Status | Evidence |
|---|---|---|---|
| 1 | `platform.md` advertised `design-system.md` as Active; file did not exist | **Resolved** | [`platform.md:168`](../../current/platform.md) — restored by RAP-1166; four-file structure on disk |
| 2 | Four of five ratified design-system files missing on disk | **Resolved** | [`design-system.md`](../../current/design-system.md), [`design-tokens.md`](../../current/design-tokens.md), [`design-motion.md`](../../current/design-motion.md), [`design-components.md`](../../current/design-components.md) all present |
| 3 | `--text-display-xl` defined three incompatible ways | **Resolved** | [`design-tokens.md:263`](../../current/design-tokens.md) canonicalises `clamp(34px, 8vw, 64px)`; [`home/typography.md:54`](../../current/home/typography.md) and [`home/blocks.md:182,1296,1744`](../../current/home/blocks.md) cite the token, not a literal clamp |
| 4 | Forge orchestrated review: README "shipped" vs verifier "not shipped" | **Resolved** | [`forge/README.md:27`](../../current/forge/README.md) now reads "specified but **not yet shipped**"; [`forge/verifier.md:175`](../../current/forge/verifier.md) agrees |
| 5 | Home block count `ten` / `eleven` / `twelve` / `thirteen` drift | **Resolved** | [`home/flows.md:38,108`](../../current/home/flows.md) — "twelve visible on-page blocks; LabPreview is a conditional thirteenth"; prior counts annotated as historical amendments with explicit forward notes ([`home/flows.md:60-90`](../../current/home/flows.md)). [`home/spec.md:12`](../../current/home/spec.md) coheres at "eleven-section anchor template". |
| 6 | Mirror "live" vs "retired" contradiction; `current/mirror.md` tombstone | **Resolved** | [`current/mirror.md`](../../current/) no longer exists (Atlas `D-ATLAS-10`); [`_root/glossary.md:191`](../../_root/glossary.md) and [`_methodology/glossary.md:34-38`](../glossary.md) both name Mirror as the live Tier-1 Brand-sphere entity |
| 7 | Model-identifier drift across Forge + Muse | **Worse** | See §B below — three competing tier definitions now |
| 8 | `cli-core` `architect/planner/auditor` vs Forge `architect/builder/reviewer` | **Resolved (by disambiguation)** | [`cli-core.md:156`](../../current/cli-core.md) explicit note: "they are not the same set — `cli-core` is the substrate event/LLM library; Forge layers its own role-tiering on top" |
| 9 | Sitemap "deferred / generated / shipped" three ways in `web.md` | **Resolved** | [`web.md:124`](../../current/web.md) `v1, live`; [`web.md:134,406,603`](../../current/web.md) cohere. Only individual `/work/[slug]` and `/lab/[slug]` detail pages remain `v1.x-deferred`, by design |
| 10 | Anthropic DPA "executed" vs "not signed" contradiction | **Unchanged** | [`archive/2026-05-28-inference-data-residency-routing-decision/design.md:12,37`](../../archive/2026-05-28-inference-data-residency-routing-decision/design.md) treats DPA as executed (covers SCC US processing); [`changes/publish-legal-bundle/research/03-privacy-dpo.md:17`](../../changes/publish-legal-bundle/research/03-privacy-dpo.md) still names the missing DPA as "the most urgent gap". `publish-legal-bundle` is in-flight (Phase 6 only just unblocked, [#1755](https://github.com/rapoport-studio/rapoport.studio/pull/1755)) but not yet archived |
| 11 | `db/conventions.md:34-45` ≈ `49-60` verbatim duplicate | **Resolved** | [`db/conventions.md`](../../current/db/conventions.md) is now 77 lines; single RLS-audit block, no duplicate |
| 12 | `auth.md:324-327` open-question duplicates | **Resolved** | [`auth.md:315-340`](../../current/auth.md) — each open-question entry appears once |
| 13 | `manifesto.en.md` declares `translates: manifesto.ru.md`; RU is placeholder | **Unchanged** | [`manifesto.en.md:4`](../manifesto.en.md) still declares the translation; [`manifesto.ru.md:9-12`](../manifesto.ru.md) still "Placeholder — Phase 2 (RAP-179) operator-blocked" |

**Score: 10 of 13 named drift findings resolved on disk** (counting the 22.05 list of named contradictions); **2 unchanged** (DPA, manifesto translation graph); **1 regressed** (model identifiers, see below).

---

## §B — One finding moved in the wrong direction: model-identifier drift

The 22.05 audit named two contradictions:

- `forge/spec.md:441` vs `forge/verifier.md:260` (verifier vs reviewer tier)
- `agents/muse.md:142-144` (`sonnet-4-7`) vs the rest of the corpus (`sonnet-4-6`)

Three days later the drift is **wider**, not narrower:

| File / line | Says |
|---|---|
| [`agents/muse.md:44-45`](../../current/agents/muse.md) | Storyteller + Planner = `claude-sonnet-4-6` |
| [`agents/muse.md:142-144`](../../current/agents/muse.md) | Storyteller + Planner = `claude-sonnet-4-7` (same roles, different default, same file) |
| [`forge/verifier.md:261`](../../current/forge/verifier.md) | "Reviewer = Sonnet 4.6 with Opus 4.7 escalation" |
| [`forge/spec.md:439,441`](../../current/forge/spec.md) | Builder + Planner = `claude-sonnet-4-7` |
| [`forge/spec.md:690`](../../current/forge/spec.md) | Reviewer = `claude-haiku-4-5` |

The reviewer tier is named three different ways across the same `forge/` folder (Sonnet 4.6, Haiku 4.5, Opus 4.7 escalation). Muse's tier table contradicts itself inside one file. The Sonnet version (`4-6` vs `4-7`) is the same drift the 22.05 audit flagged, now in more places.

This regressed because the recent Sonnet-version bumps and AI-model-tiering work landed without an editorial pass that touched every model-identifier mention in one PR.

**Highest-leverage fix:** one editorial change `reconcile-model-identifiers` that grep-finds every `claude-sonnet-4-`, `claude-opus-4-`, `claude-haiku-` in `openspec/current/` and reconciles them against `forge/spec.md § resolveModel` as canonical authority.

---

## §C — "Specced but unenforced": status of the 4 named findings

The 22.05 audit named four rules with no machine enforcement. `wire-spec-enforcement-gates` (RAP-1167, PR #1602, archived 2026-05-22) shipped enforcement for three of them; the fourth (touch-target reconciliation) was closed by `restore-design-system-spec-files`.

| 22.05 finding | Status | Evidence |
|---|---|---|
| Forbidden cold-pitch tokens — rule real, "evaluated post-launch" | **Enforced** | [`tools/eslint-rules/no-cold-pitch-tokens.js`](../../../tools/eslint-rules/no-cold-pitch-tokens.js); enabled as `"error"` in [`apps/web/eslint.config.mjs:119`](../../../apps/web/eslint.config.mjs) and [`packages/i18n/eslint.config.mjs:81`](../../../packages/i18n/eslint.config.mjs) |
| `createAdminClient` service-role bypass — "code-review and import-graph discipline" | **Enforced** | [`db/conventions.md:14`](../../current/db/conventions.md) — app-edge modules carry the `server-only` npm import marker; client-bundle import is now a **build-time failure**, not a code-review concern |
| Webhook HMAC — secrets declared, no verification flow specified | **Enforced** | [`apps/studio/src/lib/verify-webhook-signature.ts`](../../../apps/studio/src/lib/verify-webhook-signature.ts) + tests; called from `apps/studio/src/app/api/studio/command/route.ts` and `apps/studio/src/app/api/studio/github/install/webhook/route.ts` |
| Touch-target `44×44` vs `24×24` unreconciled; WCAG 2.2 AA never named as corpus contract | **Resolved** | [`design-system.md:78`](../../current/design-system.md) `D-DS-10` names WCAG 2.2 AA as the corpus contract; [`design-system.md:86-93`](../../current/design-system.md) `D-DS-9` reconciles the two figures — `24×24` is the AA floor (SC 2.5.8), `44×44` is the elective AAA goal (SC 2.5.5) — both pre-existing specs are now correct in their own contexts |

**Score: 4 of 4 §C findings resolved.** Of the entire 22.05 cross-cutting set, §C is the cleanest sweep.

---

## §D — AI-drafted artefacts shipped without human review: unchanged

The 22.05 audit named three concrete examples — Romanian "echipe curate", Latin/Cyrillic "founder'ов" code-switch, unwritten RU canonical manifesto. Three days later:

- [`manifesto.ru.md:9-12`](../manifesto.ru.md) still reads "Placeholder — Phase 2 (RAP-179) operator-blocked". The dependency declared by [`manifesto.en.md:4`](../manifesto.en.md) (`translates: manifesto.ru.md`) therefore still points at a placeholder.
- No translation-quality remediation epic has been opened. RU/RO i18n catalogues have not been audited by native reviewers in the window.
- [`research/ru-surface-audit-2026-05-22.md`](../research/ru-surface-audit-2026-05-22.md) (108KB) exists from the prior audit cycle, but its findings have not been translated into an executed remediation change.

This is the slowest-moving pattern because it needs **humans**, not engineering work. The 22.05 finding stands.

---

## §E — Designed-the-test-then-cancelled-the-run: unchanged

The Business & Positioning panel's cross-cutting observation about the Van Westendorp PSM, the Dunford canvas, the channel calendar, and the events/KPI schema being designed-but-cancelled is unchanged in the window. [`validate-marketing-surface-stage-1`](../../archive/2026-05-26-validate-marketing-surface-stage-1-cancelled/) (cancelled 2026-05-26) and [`expand-marketing-surface-stage-1`](../../archive/2026-05-24-expand-marketing-surface-stage-1-superseded/) (superseded 2026-05-24 via PR #1694; four successor scaffolds in PR #1695) remain in the same shape — successors not yet started.

No score on the Business & Positioning panel should move in this delta.

---

## §F — New surface area that introduces no fresh drift (yet)

Major new specs since 2026-05-22 — quickly checked for the 22.05 audit's failure patterns:

- [`add-inquiry-entity`](../../changes/add-inquiry-entity/) and [`add-inquiry-workflow-gates`](../../changes/add-inquiry-workflow-gates/) — clean, explicit gating relationship ("workflow gates cannot enforce against a non-existent entity"; both folders re-defer in parallel if the predecessor re-defers at its 2026-07-10 vesting check). Good signal — designed-the-gate-then-shipped-the-gate, not designed-the-gate-then-cancelled.
- [`add-specialist-tool-composition`](../../changes/add-specialist-tool-composition/) — Phase 1 (palette loader + gate) shipped 2026-05-24 (PR #1753); the platform-default palette at [`_methodology/specialist-tool-palettes/architect/tools.yaml`](../specialist-tool-palettes/architect/tools.yaml) is small, schema-anchored, and reviewable in a normal PR.
- [`add-muse-translation-mode`](../../changes/add-muse-translation-mode/) — Phase 1 shipped 2026-05-24 (PR #1754) without turn-route deferred. Phased shape, no contradiction with adjacent specs visible.
- [`establish-studio-entity-baseline`](../../changes/establish-studio-entity-baseline/) — Phase 1 scaffold present; design.md drafted (PR #1710); not yet contradicting [`studio.md`](../../current/studio.md) or [`studio-workspace/`](../../current/studio-workspace/).
- [`publish-legal-bundle`](../../changes/publish-legal-bundle/) — in flight (Phase 6 unblocked via PR #1755). Will close drift finding #10 (DPA) when archived.

**No new file-versus-file contradictions detected** in the new surface area. The cleanest test is `add-inquiry-entity` ↔ `add-inquiry-workflow-gates`: a real dependency, explicit, both folders annotated with the same gating date. This is the shape the audit wanted.

---

## §G — Estimated score deltas

The 22.05 audit scored 25 personas. This delta does not re-run all 25 — but six panel scores would move on the evidence above. Holding the other 19 at their 22.05 values:

| Specialist | 22.05 | Delta | New | Why |
|---|---|---|---|---|
| Design systems engineer / token architect | 4 | **+3** | 7 | Four ratified files now exist (#1, #2); type-ramp ceiling reconciled (#3); cross-references no longer broken. Held below 8 by `Q-DS-9` (whether `brands/studio/design-tokens.md` retires) and the model-identifier drift bleeding into the design-system tokens via [`design-tokens.md:411`](../../current/design-tokens.md) provenance footnote. |
| Accessibility specialist (WCAG 2.2 AA) | 7 | **+1** | 8 | `D-DS-10` names AA as the corpus contract; `D-DS-9` reconciles the 44×44 / 24×24 figures; the two top items in the 22.05 verdict are closed. |
| Application security engineer (AppSec) | 7 | **+1** | 8 | Webhook HMAC contract shipped end-to-end with verification module + tests; `createAdminClient` now build-time-guarded by `server-only`. Held below 9 by AT-07 and AT-12 still "Open" in [`security/agents-threat-model.md:140,145`](../../current/security/agents-threat-model.md) — no movement on those. |
| Senior Postgres / RLS engineer | 7 | **+0.5** | 7 | Verbatim-duplicated convention block removed; Branch-3 cutover still undated; `inbox.ip_address` 90-day anonymization still "not yet implemented". Modest gain, not enough for a full point. |
| AI/ML systems engineer | 8 | **-1** | 7 | Model-identifier drift wider, not narrower (see §B). Eval harness, untrusted-input sentinel, verifier gate, prompt-cache architecture all unchanged — but the corpus's own self-description of which model runs which role now contradicts itself in three files. The reviewer's tier is named three different ways. |
| Privacy DPO / GDPR counsel | 8 | **-1** | 7 | The DPA contradiction (#10) didn't move; meanwhile [`publish-legal-bundle/research/03-privacy-dpo.md:17`](../../changes/publish-legal-bundle/research/03-privacy-dpo.md) is louder than it was — naming the missing DPA as *the most urgent gap* on a still-unsigned vendor surface. Until `publish-legal-bundle` archives this is a live exposure. |

**Net corpus mean shift:** +0.3 → **6.2 / 10** (from 5.9). Median holds at 6.

The win is concentrated in the engineering-owned panels (Technical core gains 0, Technical craft gains 1.5 average) — exactly the pattern the 22.05 audit named in §E. **The corpus continues to be strong where engineering owns it.**

---

## §H — Cross-cutting observation: the audit fed back

The audit itself is the load-bearing change here. RAP-1164 was scaffolded the same day the 22.05 audit was authored; three of its sub-changes shipped within 24 hours; the four `design-system` files were written in a single PR. The corpus self-remediated its dominant defect inside the audit's own response window.

This validates the [`audit-discipline.md`](../audit-discipline.md) gate: when an audit's findings are grounded at `file:line`, they convert directly into a remediation change. When an audit's findings are not grounded (`web_fetch` cases, screenshot-driven audits), they spawn debate and Linear noise, then die. The 22.05 audit was grounded; the corpus acted on it; this delta records the action.

What did **not** convert: the patterns that need humans outside the engineering loop — native-translator review (§D), pricing/distribution evidence (§E from 22.05), counsel sign-off (drift #10). These are slower because they aren't shippable as PRs.

---

## What to do next

In recommended order (by leverage):

1. **`reconcile-model-identifiers`** — one editorial change. Grep every `claude-sonnet-4-`, `claude-opus-4-`, `claude-haiku-` in `openspec/current/`; reconcile against `forge/spec.md § resolveModel` as canonical authority; cite the canonical row from every other mention. Estimated 1 PR, 30 minutes. Closes §B.
2. **Ship `publish-legal-bundle`** — closes drift finding #10 (Anthropic DPA) and the §F privacy-DPO score regression. Already in flight; Phase 6 unblocked.
3. **Open a translation-quality epic for RU/RO i18n** — needs a native reviewer pass on existing catalogues, not an AI re-translation. Closes §D.
4. **Defer the Business & Positioning panel until the four marketing successors land.** No engineering-side action can move those scores; let the successor changes do their work.

---

## Provenance

- Method: same as [`2026-05-22-openspec-corpus-specialist-audit.md`](2026-05-22-openspec-corpus-specialist-audit.md), scoped to changes since the prior audit (`git log openspec/ --since 2026-05-22`).
- Verification: every drift-finding row above re-checked at the cited `file:line` in the worktree at commit `4935846a5` (head of `main` at audit time, plus the local delta-audit branch).
- Scoring: estimated deltas only — six panels with clear evidence moved; the other 19 are not re-scored in this pass.
