---
title: OpenSpec corpus — 25-specialist audit
status: audit
owner: pavel
audience: [pavel, ai, reviewer]
date: 2026-05-22
tldr: Twenty-five professional specialist personas each scored the OpenSpec corpus 0–10 from their discipline. Corpus mean 5.9/10 (median 6). Strongest where engineering owns it (Technical core 7.5, Legal research 6.4); weakest where it needs external evidence or human review (Business/positioning 5.0, Localization 5.25). Dominant defect across every panel: internal spec drift — file-versus-file contradictions the corpus has not reconciled.
---

# OpenSpec corpus — 25-specialist audit

> A multi-specialist audit of the Rapoport Studio OpenSpec corpus. Reference
> precedents: `changes/expand-marketing-surface-stage-1/` (marketing round 1),
> `changes/validate-marketing-surface-stage-1/` (marketing round 2),
> `changes/publish-legal-bundle/research/` (legal bundle). Method gate:
> [`audit-discipline.md`](../audit-discipline.md).

---

## Method

Twenty-five professional specialist personas were run as AI personas across six
panels. Each persona read the OpenSpec specs relevant to its discipline and
assigned **one integer score 0–10** answering: *how well is the OpenSpec work
done, judged from my discipline?*

What is being rated: the **spec corpus** — completeness, soundness, internal
consistency, defensibility of design decisions — **not** implemented code.

Grounding: this audit reads the repository directly, so it satisfies the
`audit-discipline.md` verification gate by construction. Every finding cites an
exact `file:line`. A claim that could not be grounded was dropped. Findings are
tagged `[STRENGTH]` or `[GAP]`.

Calibration: the full 0–10 range was used deliberately. Excellent + complete +
internally consistent → 8–9. Solid with real gaps → 6–7. Thin / stub /
contradictory → 3–5. No score was parked at a comfortable 7.

---

## Master scorecard

| # | Panel | Specialist | Score |
|---|---|---|---|
| 1 | Technical core | Senior Next.js / edge-runtime architect | **8** |
| 2 | Technical core | Senior Postgres / RLS engineer | **7** |
| 3 | Technical core | Application security engineer (AppSec) | **7** |
| 4 | Technical core | AI/ML systems engineer | **8** |
| 5 | Technical craft | Design systems engineer / token architect | **4** |
| 6 | Technical craft | Accessibility specialist (WCAG 2.2 AA) | **7** |
| 7 | Technical craft | DevEx / platform engineer | **7** |
| 8 | Product & Design | Product Designer / Information Architect | **6** |
| 9 | Product & Design | Brand strategist + identity-system designer | **5** |
| 10 | Product & Design | Type designer / typographer | **4** |
| 11 | Product & Design | Qualitative UX researcher (Switch / JTBD) | **7** |
| 12 | Business & Positioning | Category-design strategist (Dunford) | **6** |
| 13 | Business & Positioning | B2B pricing strategist | **4** |
| 14 | Business & Positioning | DevRel / technical distribution strategist | **3** |
| 15 | Business & Positioning | B2B PMM + CRO specialist | **7** |
| 16 | Business & Positioning | Marketing analytics / measurement designer | **5** |
| 17 | Legal & Compliance | Commercial lawyer (Romania / Moldova) | **8** |
| 18 | Legal & Compliance | Privacy DPO / GDPR counsel | **8** |
| 19 | Legal & Compliance | AI governance counsel | **7** |
| 20 | Legal & Compliance | IP / open-source lawyer | **4** |
| 21 | Legal & Compliance | Israeli privacy + commercial lawyer | **5** |
| 22 | Content & Localization | B2B copywriter, native English | **7** |
| 23 | Content & Localization | Native Russian copywriter + editor | **4** |
| 24 | Content & Localization | Native Romanian copywriter + editor | **4** |
| 25 | Content & Localization | Native Hebrew copywriter | **6** |

**Corpus mean: 5.9 / 10. Median: 6.**

### Panel rollup

| Panel | Specialists | Mean |
|---|---|---|
| Technical core | 4 | **7.5** |
| Legal & Compliance | 5 | **6.4** |
| Technical craft | 3 | **6.0** |
| Product & Design | 4 | **5.5** |
| Content & Localization | 4 | **5.25** |
| Business & Positioning | 5 | **5.0** |

### Score distribution

| Score | Count | Specialists |
|---|---|---|
| 8 | 4 | Next.js architect, AI/ML engineer, Commercial lawyer, Privacy DPO |
| 7 | 8 | Postgres/RLS, AppSec, Accessibility, DevEx, JTBD researcher, PMM+CRO, AI governance, EN copywriter |
| 6 | 3 | Product Designer/IA, Category-design strategist, HE copywriter |
| 5 | 3 | Brand strategist, Marketing analytics, Israeli lawyer |
| 4 | 6 | Design systems, Typographer, Pricing strategist, IP/OSS lawyer, RU copywriter, RO copywriter |
| 3 | 1 | DevRel strategist |

---

## Cross-cutting findings

Five patterns recurred across panels that no single specialist owns.

### A. Internal spec drift is the corpus's dominant defect

Nearly every panel flagged file-versus-file contradictions the corpus has not
reconciled. These need no new decisions — only one editorial pass:

- `platform.md:87,160,507` advertise `openspec/current/design-system.md` as an
  **Active capability**; the file does not exist, and four of the five ratified
  design-system files are absent on disk (`design-system-convention.md:23-77`).
- Forge orchestrated review: `forge/README.md:27` — *"Orchestrated review is
  shipped"* — vs `forge/verifier.md:163,175` — *"code path not yet shipped …
  will fail until the implementation lands"*. Same capability, opposite claims,
  same folder. (Verified during this audit.)
- `--text-display-xl` is defined three incompatible ways:
  `design-tokens.md:260` (fixed 64px), `home/typography.md:54`
  (`clamp(34px,8vw,64px)`), `home/blocks.md` Hero contract
  (`clamp(40px,6vw,64px)`).
- Home block count: `home/spec.md:12` promises "ten-section template" then
  lists eleven; `home/flows.md:59,73,87` carries thirteen / twelve / eleven in
  stacked amendments.
- "Mirror": `_root/glossary.md:187-191` defines it as a *live* concept;
  `_methodology/glossary.md:32-37` says it is a *"retired term"*.
  `current/mirror.md:1-12` is a tombstone, yet `openspec/mirror/*/perception.md`
  hold live files.
- Model-identifier drift: `forge/spec.md:441` (`verifier` = Opus 4.7) vs
  `forge/verifier.md:260` (Reviewer = Haiku→Opus); `agents/muse.md:142-144`
  (`sonnet-4-7`) vs the rest of the corpus (`sonnet-4-6`).
- `cli-core.md:130-152` `EventKind` enum and `architect|planner|auditor` roles
  vs Forge's actual `architect|builder|reviewer` tiering (`verifier.md:261`) —
  the declared "single source of truth" contradicts its primary consumer.
- Sitemap stated three ways in `web.md`: deferred (`:114`), generated (`:374`),
  shipped (`:570`).
- Anthropic DPA: `inference-data-residency-routing-decision/design.md:12` treats
  it as executed; `publish-legal-bundle/research/03-privacy-dpo.md:17` says it
  is not.
- Verbatim-duplicated text: `db/conventions.md:34-45` ≈ `49-60`;
  `auth.md:324-327` (two open-question pairs duplicated).
- False translation graph: `manifesto.en.md:4` declares `translates:
  manifesto.ru.md`, but the RU file is a placeholder (`manifesto.ru.md:9-12`).

### B. "Designed the test, then cancelled the run"

The Business & Positioning panel's cross-cutting observation: the corpus
*designs* validation work to a high standard — a Van Westendorp PSM protocol
(`validate-marketing-surface-stage-1/design.md:121-125`), a Dunford categorical
canvas, a 12-week channel calendar (`design.md:181-185`), an events/KPI schema
(`design.md:208-215`) — and then **cancels or backlogs four of the five research
streams** via the right-sizing Outcome-2 decision
(`validate-marketing-surface-stage-1/proposal.md:25-56`). Both marketing changes
are additionally FROZEN. Net effect: positioning, pricing, distribution, and
measurement are founder-asserted, not evidenced. A stale gate compounds it —
Q-POS-2 is chained to `add-funding-strategy`, which already archived 2026-05-13
(`positioning-decision-framework.md:296-302` vs `funding-strategy.md:21`).

### C. Specced but unenforced

- Forbidden cold-pitch tokens: rule is real (`platform.md:549-558`) but
  enforcement is "evaluated post-launch" (`platform.md:564`) — no CI gate.
- `createAdminClient` service-role bypass relies on "code-review and
  import-graph discipline" (`db/conventions.md:14`); the spec names three
  enforcement options and picks none.
- Webhook HMAC: `config.md:61` declares `GITHUB_WEBHOOK_SECRET` /
  `LINEAR_WEBHOOK_SECRET`, but no signature-verification flow is specified
  anywhere.
- Touch-target floor: `44×44` (`home/spec.md:66`) vs `24×24` per WCAG 2.2
  SC 2.5.8 (`chat-redesign-a11y.md:71`) — unreconciled; and "WCAG 2.2 AA" is
  never named as a corpus-wide conformance contract.

### D. AI-drafted artefacts shipped without human review

The RAP-906 lesson is recurring. RU/RO i18n catalogs contain the false-friend
"echipe curate" (Romanian *curat* = clean, not curated) and the
Latin/Cyrillic code-switch "founder'ов"; the RU canonical manifesto is unwritten
by design (`manifesto.ru.md:9-12`); Hebrew chrome was shipped and pulled the
same day (`platform.md:541`).

### E. Strong where engineering owns it, weak where it needs evidence or humans

Technical core (7.5) and Legal research (6.4) lead; Business/positioning (5.0)
and Localization (5.25) trail. The corpus writes superb architecture and is
unusually honest in self-documentation — the OpenNext upstream-bug register
(`frontend-runtime.md:520-545`), the 15-threat STRIDE/ATLAS model
(`security/agents-threat-model.md:100-148`), the verifier's own history notes
(`verifier.md:36-52`). It stalls where the work needs external input it cannot
generate itself: pricing data, user interviews, native translators, counsel
sign-off, channel validation.

---

## Panel detail

### Panel 1 — Technical core

#### Senior Next.js / edge-runtime architect — 8/10

- [STRENGTH] The closure-capture tenant-leak hazard in Next 16 `'use cache'` is
  identified precisely and made enforceable with three layers (ESLint rule,
  Forge reviewer rule, mandatory tenant tag segment) — `frontend-runtime.md:120-131`.
- [STRENGTH] The silent-no-op failure of `'use cache'` without a backing store
  is named "the most expensive failure mode in this architecture" and gated by
  a CI smoke test — `frontend-runtime.md:265-273`.
- [STRENGTH] HOT-tier rendering correctly forbids serving HOT data from the RSC
  payload because of the 30s router-cache floor — `frontend-runtime.md:401-407`.
- [GAP] `web.md:624` says `experimental.staleTimes` is set on
  `apps/web/next.config.ts`, but `frontend-runtime.md:585` says `apps/web` ships
  only static routes — the setting is inert and undocumented as such.
- [GAP] Sitemap status disagrees across three sections — `web.md:114-115`
  (v1.x-deferred), `web.md:374-379` (generated), `web.md:570-572` (shipped).
- [GAP] Seven launch-blocking open questions in `web.md:531-540` have no
  decision owner or trigger date.

**Verdict:** A genuinely strong, professionally-authored edge-runtime contract;
the OpenNext gotchas and upstream-bug register are institutional knowledge most
teams never write down. Held below 9 by stale cross-section drift. Highest-
leverage fix: one editorial pass reconciling sitemap status and the `apps/web`
router-cache claim.

#### Senior Postgres / RLS engineer — 7/10

- [STRENGTH] The three-branch RLS chain (platform_owner short-circuit →
  project_members EXISTS → legacy canvas_invitees) is fully specified with
  canonical SQL, mandatory branch ordering, and USING/WITH-CHECK role-list
  discipline — `db/entities.md:1195-1234`, `db/conventions.md:64-83`.
- [STRENGTH] Index coverage is deliberate — `project_members_active_idx ON
  (project_id, user_id) WHERE accepted_at IS NOT NULL AND revoked_at IS NULL`
  is exactly the proxy lookup predicate — `db/entities.md:1141`.
- [STRENGTH] `bootstrap_platform_owner` uses `BEFORE INSERT` with an explicit
  rationale (JWT signed at session creation) — `db/entities.md:343-364`.
- [GAP] The RLS adversarial-audit convention block is duplicated verbatim —
  `db/conventions.md:34-45` and `:49-60` are byte-identical.
- [GAP] `inbox.ip_address` 90-day anonymization relies on "a scheduled task
  (not yet implemented)" — a GDPR Art. 5(1)(e) obligation with no implementation
  — `db/entities.md:141-143`.
- [GAP] Legacy Branch-3 RLS cutover is prose-only with no owner —
  `db/entities.md:1242-1244`; an indefinitely-deferred third RLS branch is
  permanent attack surface.

**Verdict:** The project-membership-first RLS redesign is the corpus's strongest
schema work. Held at 7 by a verbatim-duplicated block, an unimplemented GDPR
retention job, and an undated legacy-branch cutover. Highest-leverage fix:
delete the duplicate block and assign an owner + date to the Branch-3 cutover.

#### Application security engineer (AppSec) — 7/10

- [STRENGTH] Cookie posture fully specified — `httpOnly`, `sameSite=lax`,
  `secure` in prod, `Domain` production-only so dev avoids cross-origin breakage
  — `auth.md:175-183`.
- [STRENGTH] The token-hash magic-link flow type-whitelists the `type` param and
  same-origin-strict-validates the `next` redirect — `auth.md:79`,
  `linkable-ui.md:367`.
- [STRENGTH] The agent-layer threat model is real security work — 15
  STRIDE/ATLAS threats, L×I scoring, honest "Open" markers —
  `security/agents-threat-model.md:100-148`.
- [GAP] No webhook HMAC verification is specified — `config.md:61` declares the
  webhook secrets but no verification contract exists.
- [GAP] AT-07 (secret leakage into Anthropic prompt logs) and AT-12
  (`commit_file` path traversal) are the joint-highest live threats and both
  remain "Open — needs OpenSpec change" — `agents-threat-model.md:140,145`.
- [GAP] `auth.md:324-327` duplicates two full open-question entries verbatim.

**Verdict:** Auth/cookie/magic-link surface solidly specified; the threat model
is a credit to the corpus. Held at 7 by an absent webhook-HMAC contract and two
top-RPN threats unscheduled. Highest-leverage fix: write the GitHub/Linear
webhook HMAC-verification contract.

#### AI/ML systems engineer — 8/10

- [STRENGTH] The Muse Architect eval harness is real, not aspirational —
  `tools/muse-evals/` has `judge.ts`, golden-label calibration fixtures, a CI
  regression gate with a frozen baseline and a hard rule against bundling
  baseline updates with prompt edits — `muse.md:236-256`.
- [STRENGTH] The untrusted-input sentinel convention is precise and complete —
  tag shape, trust-classification table, nested-escape rule, canonical refusal
  text — `security/untrusted-input-convention.md:11-57`.
- [STRENGTH] The Forge verifier gate — five programmatic checkpoints,
  content-based (not `is_error`-based) failure-marker matching, the RAP-740
  reviewer context-isolation fix — `verifier.md:14-50,94,233-253`.
- [STRENGTH] The 4-layer prompt-cache architecture is correctly tiered with a
  cache-hit-ratio observability rule — `forge/spec.md:524-558,705-707`.
- [GAP] Model-tiering contradiction: `forge/spec.md:441` lists
  `personas/verifier.ts` as Opus 4.7 while `forge/verifier.md:260` says Reviewer
  defaults to Haiku — verifier vs reviewer conflated.
- [GAP] Tier-2 tool cost is silently uncounted — `domain_research.execute`
  bypasses `executeToolWithGuards` — `muse.md:395`.

**Verdict:** The most mature slice of the corpus. Held below 9 by
model-identifier drift and an acknowledged cost-guard blind spot. Highest-
leverage fix: reconcile verifier/reviewer model tier and normalize the Sonnet
version string corpus-wide.

### Panel 2 — Technical craft

#### Design systems engineer / token architect — 4/10

- [GAP] The capability map advertises `openspec/current/design-system.md` as
  "Active", but the file does not exist — `platform.md:87,160`; the same dead
  path is cited as the home for severity tokens — `platform.md:507`.
- [GAP] The ratified five-file structure is fully specified, yet four of five
  files are missing on disk — `design-system-convention.md:23-77`; `ui.md`
  links to these non-existent siblings ~30 times — `ui.md:10,506-509`.
- [GAP] `ui.md` delegates token *values*, motion, and component anatomy out of
  itself by R10 authority — so with the siblings absent, the token taxonomy,
  `$type` registry, contrast and motion contracts have no spec home that exists
  — `ui.md:46,75,508`.
- [STRENGTH] The convention itself is excellent — W3C DTCG `$type` discipline,
  IBM Carbon two-mode motion, `--duration-scalar` reduced-motion mechanism,
  closed duration/easing scales — `design-system-convention.md:107-201`.
- [STRENGTH] Operating model and audit protocol are mature — four bounded roles,
  eight-step workflow, S0/S1/S2 severity — `design-system-operating-model.md:9-49`.
- [GAP] The Q2 audit is a non-audit — one out-of-repo finding, and it admits
  "the five audit dimensions were not exercised" —
  `2026-Q2-design-system-audit.md:30-31`.

**Verdict:** An excellent *convention* sitting atop a *corpus that does not
exist*. Highest-leverage fix: author (or restore) `design-tokens.md`,
`design-motion.md`, `design-components.md`, `design-system.md` — until then
every `ui.md` cross-reference is a broken link.

#### Accessibility specialist (WCAG 2.2 AA) — 7/10

- [STRENGTH] `forms.md` Rule 4 is a textbook accessible-error contract —
  `aria-invalid`, appending `aria-describedby`, `role="alert"` only on
  dynamically-appearing containers — `forms.md:49-57`.
- [STRENGTH] The chat SR runbook is a 36-cell manual matrix with expected
  NVDA/VoiceOver utterances complementing an automated axe gate —
  `chat-sr-runbook.md:49-130,238`.
- [GAP] "WCAG 2.2 AA" is never named as a corpus-wide contract — only ad-hoc
  per-criterion citations; the convention's a11y rule R9 names no conformance
  target — `design-system-convention.md:222`.
- [GAP] Touch-target contracts conflict — `44×44` (`home/spec.md:66,75`) vs
  `24×24` per SC 2.5.8 (`chat-redesign-a11y.md:71`); `44px` is the AAA-level
  2.5.5.
- [STRENGTH] Reduced-motion handling is consistently specified —
  `home/blocks.md:76,212,348,494`.
- [GAP] The streaming live-region story is split — research says "announce only
  on completion" (`chat-redesign-a11y.md:14`) but the runbook describes
  streaming into a `polite` region directly (`chat-sr-runbook.md:96-99`).

**Verdict:** Strong, well-cited per-surface accessibility specs undermined by no
single named "WCAG 2.2 AA" contract and a real 44px-vs-24px contradiction.
Highest-leverage fix: one decision record naming WCAG 2.2 AA as the studio-wide
contract and reconciling target-size to SC 2.5.8's `24×24` minimum.

#### DevEx / platform engineer — 7/10

- [STRENGTH] The 18-command Forge set is coherent and exhaustively
  path-anchored — `commands.md:16,98-116,121-131`.
- [GAP] Direct contradiction: `verifier.md:175` lists orchestrated review as
  Known-gap item 8, "specified but unimplemented", while `README.md:27` states
  "Orchestrated review is shipped." (Verified during this audit.)
- [STRENGTH] The verifier spec is honest about its own history — the
  `--auto-plan` flag split, "five not six checkpoints", marker-based
  verification — `verifier.md:36-52,94`.
- [STRENGTH] Build budgets and observability are real — `budgets.build` USD cap,
  zombie-build watchdog, `TickDecision` JSON logs, 60% coverage gate —
  `commands.md:230-254`, `verifier.md:165`.
- [GAP] `cli-core.md`'s `EventKind` enum (5 constants) and
  `architect|planner|auditor` roles do not match Forge's
  `architect|builder|reviewer` — `cli-core.md:130-139,80` vs `verifier.md:261`.
- [GAP] Two model-routing tables disagree — `cli-core.md:148-152` (`planner →
  sonnet-4-6`) vs `verifier.md:261` — and `cli-core` is the declared single
  source of truth — `cli-core.md:213`.

**Verdict:** A genuinely strong, refreshingly honest DevEx corpus held back by a
flat shipped/unshipped contradiction and substrate-vs-consumer drift.
Highest-leverage fix: reconcile `README.md:27` with `verifier.md:175` and align
`cli-core`'s enums with Forge's real role tiering.

### Panel 3 — Product & Design

#### Product Designer / Information Architect — 6/10

- [STRENGTH] The eleven-section block template is disciplined — every block
  scans the same 10 anchors (`home/spec.md:11-25`), bound to component path and
  spec dependencies (`spec.md:31-43`).
- [STRENGTH] Page invariants are explicit — one primary CTA, audience tagging,
  no mode inversion, unmount-not-placeholder — `home/flows.md:33-40`.
- [GAP] Block count is a documented mess — `spec.md:12` promises "ten-section"
  then lists eleven; `flows.md:59,73,87` carries 13 / 12 / 11.
- [GAP] The current spec contradicts the proposed fix with no cross-reference —
  `flows.md:25` still describes `WhoWeWorkWith`/`BuildOnThePlatform` as live;
  `fix-home-trust-blocks-and-narrative/proposal.md:87` removes both.
- [GAP] The fix proposal is six open questions wide and has no Linear epic —
  `proposal.md:5`.
- [STRENGTH] `fix-home-trust-blocks-and-narrative` correctly diagnoses the IA
  pathology (three overlapping audience taxonomies, services at position 10) —
  `proposal.md:30-41`.

**Verdict:** The block-contract machinery is excellent; the page IA is
mid-rewrite and the corpus has not been kept honest about it. Highest-leverage
fix: stamp `home/spec.md` and `flows.md` with an "amendment pending" banner and
reconcile the block-count drift to a single number.

#### Brand strategist + identity-system designer — 5/10

- [STRENGTH] The Brand-as-umbrella ontology is well-defined — Brand = Identity +
  Design System + voice + surfaces, Identity a strict subset —
  `_root/glossary.md:155-159`.
- [GAP] The two glossaries flatly contradict each other on "Mirror" —
  `_root/glossary.md:187-191` (live concept) vs `_methodology/glossary.md:32-37`
  ("retired term").
- [GAP] `current/mirror.md` is a tombstone yet `openspec/mirror/*/perception.md`
  hold live files and `_root/glossary.md:189` reuses "Mirror" for a new meaning.
- [GAP] The studio's own brand artefact is unpopulated —
  `brands/studio/identity.md:6-11` has placeholder colour, typography, empty
  vocabulary.
- [STRENGTH] The Cast layer is a clean brand-boundary artefact — six agents,
  "team-internal only — never appear in public branding" —
  `manifesto.en.md:38-51`.
- [STRENGTH] The studio-vs-Pavel boundary is enforced structurally —
  `platform.md:44`, `positioning.md:358`.

**Verdict:** The vocabulary theory is strong but the corpus has not converged —
"Mirror" carries three live meanings the system claims to have retired.
Highest-leverage fix: reconcile the Mirror definition across both glossaries and
either repopulate or rename `openspec/mirror/`.

#### Type designer / typographer — 4/10

- [GAP] `--text-display-xl` has two incompatible definitions —
  `design-tokens.md:260` (fixed 64px) vs `home/typography.md:54`
  (`clamp(34px,8vw,64px)`).
- [GAP] A third clamp value — `blocks.md` Hero contract `clamp(40px,6vw,64px)` —
  disagrees on min and vw rate.
- [GAP] The Hero eyebrow contract specifies `text-[10px]` (`blocks.md:178`)
  while `typography.md:56-58` mandates a 12px floor and explicitly forbids
  `text-[10px]`.
- [GAP] No multi-script support is specified — font stacks at
  `design-tokens.md:273-275` carry no Cyrillic subset note and no Hebrew face,
  despite `ru`/`ro` shipping and Hebrew discussed.
- [STRENGTH] The static type ramp is coherent and implementable —
  `design-tokens.md:257-267`.
- [STRENGTH] Casing rules are precise and enforceable —
  `design-tokens.md:293-296`.
- [GAP] Vertical rhythm is asserted but not specified — `ui.md:322` defers to
  plugin defaults.

**Verdict:** The static type ramp is solid, but the fluid-display layer is
internally contradictory across three files. Highest-leverage fix: pick one
canonical `--text-display-xl` clamp, write it in `design-tokens.md` only, and
reconcile the eyebrow size to the 12px floor.

#### Qualitative UX researcher (Switch / JTBD) — 7/10

- [STRENGTH] The JTBD method spec is rigorous — `validate-marketing-surface-
  stage-1/design.md:91` names the Switch sequence and cites the
  Klement/Spiek/Christensen school.
- [STRENGTH] The forbidden-token list is disciplined and evidence-grounded — six
  banned tokens traced to a specific ethnographic read — `platform.md:551-558`.
- [GAP] The forbidden-token list has no enforcement — `platform.md:564` ("ESLint
  rule … evaluated post-launch").
- [STRENGTH] The research framework is honestly self-correcting — a right-sizing
  supersession banner and a synthetic-personas fallback —
  `validate-marketing-surface-stage-1/design.md:5,91`.
- [GAP] The persona corpus is thin where it matters — `platform.md:522`
  references F1–F5 but detail lives only in an archived research file.
- [GAP] `13-schools-of-thought.md:49` flags Research & Discovery as a "top-3
  forgotten school" — an accurate self-diagnosis not yet acted on.

**Verdict:** Where JTBD work exists it is methodologically sound and
buyer-language-grounded. Held back because the rigor lives in change/archive
documents while steady-state `current/` personas are a thin table.
Highest-leverage fix: lift F1–F5 job statements into `platform.md § Personas`.

### Panel 4 — Business & Positioning

#### Category-design strategist (Dunford) — 6/10

- [STRENGTH] The competitive-alternatives framing is Dunford-grade — a 5-row
  adjacent-category table with "why we are not them" reasoning —
  `positioning.md:41-49`.
- [GAP] Two contradictory category claims coexist unreconciled —
  `positioning.md:31` ("canvas studio for product architects", blue-ocean) vs
  `competitive-positioning-sdd-frameworks.md:46-49` ("the gap between SDD tools
  and methodology consultancies", named rivals).
- [GAP] The sharpest point-of-view doc is permanently caveated `status:
  hypothesis` — `positioning.md:3-5,21` — barred from use on the public surface.
- [GAP] The Dunford research stream was backlogged, not run —
  `validate-marketing-surface-stage-1/proposal.md:30,48`.
- [STRENGTH] `sdd-positioning.md:49-69` is a model of disciplined honesty —
  states the differentiator is "narrower than target-state specs".

**Verdict:** More category-design rigor than most early-stage studios, but it
carries two unreconciled category theses and backlogged the one stream that
would adjudicate. Highest-leverage fix: run the Dunford categorical canvas and
ratify a single category in a decision row.

#### B2B pricing strategist — 4/10

- [STRENGTH] The pricing-stack design is structurally sound — four orthogonal
  layers × three disclosure levels with anti-race-to-bottom mechanisms —
  `specialist-network.md:441-449,492-518`.
- [GAP] The central pricing decision is unresolved and chained to a satisfied
  dependency — Q-POS-2 is gated on `add-funding-strategy`
  (`positioning-decision-framework.md:296-302`), which already archived
  2026-05-13 (`funding-strategy.md:21`).
- [GAP] Every price is asserted, not evidenced — `specialist-network.md:436`
  ("placeholders"), `positioning.md:166-167` ("indicative"). No Van
  Westendorp/PSM data exists.
- [GAP] The PSM stream was designed but never executed —
  `validate-marketing-surface-stage-1/design.md:121-125` specs a competent
  protocol; the change is FROZEN.
- [GAP] The €5k Discovery price — the only hard-priced tier — has no recorded
  derivation, never sensitivity-tested.

**Verdict:** The pricing *architecture* is thoughtful; pricing *substance* is
intuition wearing a decision-framework costume. Highest-leverage fix: close the
stale Q-POS-2 gate and either commission the PSM stream or explicitly label the
floors as founder-judgment.

#### DevRel / technical distribution strategist — 3/10

- [GAP] The entire distribution plan is "publish article → wait for HN" —
  `expand-marketing-surface-stage-1/design.md:373` is the sole cross-post
  mechanic. No channel calendar, no newsletter, no Discord.
- [GAP] The DevRel research stream was cancelled outright —
  `validate-marketing-surface-stage-1/proposal.md:30,49` (graded PASS-0). The
  12-week channel calendar specced at `design.md:181-185` was never produced.
- [GAP] Cross-posting policy is an open question with a one-line founder default
  — `expand-marketing-surface-stage-1/proposal.md:194` (Q-EM-7).
- [GAP] The open-core public-mirror — the studio's whole distribution thesis —
  has no engagement loop — `open-source-positioning.md:343-388`.
- [STRENGTH] The robots.ts AI-crawler allowlist is a real, current GEO move —
  `expand-marketing-surface-stage-1/design.md:64-88`.

**Verdict:** The weakest slice. The studio recognised the gap and then cancelled
the stream meant to fix it; ICP density per channel is unmodeled. Highest-
leverage fix: un-cancel the DevRel stream — for a one-person-plus-AI studio, a
wrong primary channel costs the whole launch quarter.

#### B2B PMM + CRO specialist — 7/10

- [STRENGTH] Funnel friction is specced with evidence — multi-step intake cites
  field-count conversion data (13.85% vs 4.53% at >7 fields) —
  `expand-marketing-surface-stage-1/proposal.md:102`, `design.md:432-440`.
- [STRENGTH] The `<DecisionRouter />` self-qualification and the
  qualification-gated Stripe mechanic are genuine CRO craft —
  `design.md:408-428,462-466`.
- [STRENGTH] Founder-track vs client-track segmentation is structurally enforced
  — `home/flows.md:24,30-35`.
- [GAP] The whole conversion phase (Phase 5) is doubly gated and FROZEN —
  depends on Q-POS-1..7 (`proposal.md:96`), carve-out at `design.md:537-539`.
- [GAP] Intake-form abandonment tracking is explicitly deferred — `web.md:539` —
  the funnel ships with no instrument to diagnose where it leaks.

**Verdict:** The funnel is well-specced — multi-step intake, decision router,
qualification gate, all backed by citations. Held back because the conversion
phase is frozen behind pricing gates and ships without abandonment
instrumentation. Highest-leverage fix: unfreeze the conversion mechanics that do
not depend on pricing and ship them with field-level abandonment tracking.

#### Marketing analytics / measurement designer — 5/10

- [STRENGTH] The measurement design brief is excellent — a real leading/lagging
  KPI hierarchy and a unit-economics model —
  `validate-marketing-surface-stage-1/design.md:208-215`.
- [GAP] That brief was never executed — Stream #5 was cancelled
  (`validate-marketing-surface-stage-1/proposal.md:30,50`).
- [GAP] The marketing surface ships measurement-blind — the round-2 review
  itself diagnosed "zero events, zero KPIs, zero leading indicators"
  (`proposal.md:69`) and then cancelled the stream meant to close it.
- [STRENGTH] The cookieless posture is rigorous and well-documented —
  `platform.md:655-667`.
- [GAP] Cookieless rigor is not reconciled with measurement need — `web.md:536`
  permanently bans non-essential identifiers, so the lagging KPIs have no
  instrument and no resolution path.
- [STRENGTH] The validation-plan decision matrix is genuinely rigorous —
  `validation-plan-q3-2026.md:228-241` — but dormant (`:1-5,15`).

**Verdict:** The studio knows how to design measurement but has not specced it
for the surface it is actually shipping. Highest-leverage fix: lift the
cancelled Stream #5 events schema into a real `current/` capability spec; server-
side Cloudflare Logpush can carry the leading KPIs without a CMP banner.

### Panel 5 — Legal & Compliance

#### Commercial lawyer (Romania / Moldova) — 8/10

- [STRENGTH] The 24-section ToS skeleton is counsel-ready — load-bearing clauses
  ship as actual draft text — `publish-legal-bundle/research/01-commercial-
  lawyer.md:25-50`.
- [STRENGTH] Engagement formation treats Telegram/WhatsApp intake as legally
  binding and fixes it with an integration clause + "Order Confirmation" moment
  — `research/01-commercial-lawyer.md:64-68,169-177`.
- [STRENGTH] IP allocation uses present-tense assignment and rejects US
  work-for-hire as unreliable in MD/EU — `research/01-commercial-lawyer.md:76,183`.
- [GAP] The SOW / engagement-contract template — the document that binds a
  paying client — is not drafted anywhere — `publish-legal-bundle/proposal.md:81`.
- [GAP] No E&O insurance exists, yet the corpus carries uncapped GDPR Art. 28
  exposure into client DPAs — `design.md:65`, `research/01-commercial-lawyer.md:229`.

**Verdict:** Strong, well-sourced commercial drafting a Moldovan counsel can
review in one pass. Held below 9 because the SOW/engagement-contract template
does not exist as a draft. Highest-leverage fix: draft that template before
Phase 5 counsel review.

#### Privacy DPO / GDPR counsel — 8/10

- [STRENGTH] The tri-document architecture (ToS / Privacy Policy / per-engagement
  DPA) is correctly reasoned — `publish-legal-bundle/research/03-privacy-dpo.md:24-79`.
- [STRENGTH] The controller/processor map covers 14 data flows with lawful basis
  per flow and correctly identifies Telegram/WhatsApp as joint-controller
  surfaces — `research/03-privacy-dpo.md:91-114`.
- [STRENGTH] The Art. 13 layered-notice design for intake bases processing on
  Art. 6(1)(f) legitimate interest, not a fake consent checkbox —
  `add-intake-privacy-disclosure/design.md:11-22`.
- [GAP] The residency change picks `inference_geo: "us"` reasoning it is
  "covered by … our existing Anthropic DPA"
  (`inference-data-residency-routing-decision/design.md:12`) — but the DPO
  research says no Anthropic DPA is yet executed (`research/03-privacy-dpo.md:17`).
- [GAP] Routing EU-client data to US infrastructure by default, with no
  per-tenant EU-residency option, is deferred to a future decision record —
  `inference-data-residency-routing-decision/design.md:12,46`.

**Verdict:** The most mature legal slice — architecture, role mapping, Art. 13
design are counsel-ready. Held at 8 by one real contradiction on DPA-execution
status. Highest-leverage fix: reconcile the DPA status across both changes and
ratify the inference-geo value before counsel review.

#### AI governance counsel — 7/10

- [STRENGTH] The deployer-vs-provider analysis is rigorous — Article 3 tests run
  line by line, AI-14 reserves a role-escalation clause —
  `publish-legal-bundle/research/02-ai-governance-counsel.md:255-264,163-165`.
- [STRENGTH] The hallucination disclaimer is drafted to survive *Moffat*-style
  scrutiny — `research/02-ai-governance-counsel.md:82-101,187-200`.
- [STRENGTH] Article 50 (in force 2026-08-02) is correctly split into
  studio-to-client and client-to-end-user disclosure —
  `research/02-ai-governance-counsel.md:128-136`.
- [GAP] Article 4 AI-literacy compliance depends on an internal AI-usage policy
  at v1.0 — still v0.1 — `research/02-ai-governance-counsel.md:284-291`.
- [GAP] `risk-taxonomy.md:40` mandates "transparency labeling ('Drafted by
  Maestro')" but the legal-bundle research concludes the human-review exemption
  likely removes that obligation — the two corpora are unreconciled.
- [GAP] `red-lines.md:15-23` contains no AI Act / Art. 50 entry — the
  AI-governance posture lives only in an in-flight change.

**Verdict:** Excellent research file, correct Art. 50/Art. 4 timing. Held below
8 by the `risk-taxonomy.md` labeling contradiction and a posture resting on
operational documents that do not yet exist. Highest-leverage fix: reconcile
`risk-taxonomy.md` with the legal-bundle Art. 50 analysis.

#### IP / open-source lawyer — 4/10

- [STRENGTH] The open-source-positioning research is a thoughtful layer-by-layer
  analysis with realistic license mappings (CC-BY-SA / MIT / FSL) —
  `open-source-positioning.md:99-108,160-235`.
- [STRENGTH] The specialist-network NDA architecture is sound — two-layer, all
  signed with the studio entity, IP-assignment flowing to the studio —
  `specialist-network.md:83-98`.
- [GAP] The public mirror is shipping with no LICENSE file and no license
  declaration anywhere — `public-mirror-README-template.md:1-36`,
  `.publish-manifest.yml:1-13`.
- [GAP] `.publish-manifest.yml` publishes `_methodology/**`, all of `archive/`
  and `changes/` with no license field and a purely path-based exclude list —
  `.publish-manifest.yml:2-13`.
- [GAP] "Rapoport Studio" trademark protection is entirely absent — no
  registration plan, no usage policy — `open-source-positioning.md:446-447`.
- [GAP] The contributor-IP path is half-built — no inbound CLA/DCO for the
  public mirror — `specialist-network.md:93`, `open-source-positioning.md:228-235`.

**Verdict:** The thinking is strong but the work is research-only — the
dependency files do not exist. A public mirror shipping without a LICENSE is a
concrete legal defect. Highest-leverage fix: make the license decision (the
research already recommends CC-BY-SA / MIT / FSL) and add LICENSE files + a
trademark-reservation notice before the mirror goes public.

#### Israeli privacy + commercial lawyer — 5/10

- [STRENGTH] PPL Amendment 13 (in force 2025-08-14) is accurately summarised —
  online identifiers as personal data, the PPO concept, stricter cross-border
  rules — `research/03-privacy-dpo.md:238-246`, `cookie-law-jurisdictions.md:163-182`.
- [STRENGTH] The IL→MD cross-border problem is correctly identified as the acute
  exposure — `research/03-privacy-dpo.md:246`.
- [GAP] Despite "acute", no DPA clause, no Israeli-client addendum, no
  SCC-equivalent template is drafted — `research/03-privacy-dpo.md:199`.
- [GAP] The Israeli surface is mentioned but never specced — `platform.md:541`;
  no IL-specific legal artefact exists.
- [GAP] The intake privacy disclosure ships only en/ru/ro — no Hebrew variant,
  no Israeli-law disclosure path — `add-intake-privacy-disclosure/design.md:75-79`.
- [GAP] The D-LB-7 counsel gate engages only Moldovan + EU counsel, no IL
  counsel — `design.md:114-117`.

**Verdict:** The Israeli dimension is competently researched but the least
operationalised slice — every Israel finding lives as a flagged risk, never a
drafted clause. Highest-leverage fix: add an Israeli privacy counsel to the
D-LB-7 gate and draft an IL-client DPA addendum.

### Panel 6 — Content & Localization

#### B2B copywriter, native English — 7/10

- [STRENGTH] The cold-pitch forbidden-token list is unambiguous — six tokens,
  surfaces, and rationale — `platform.md:549-558`.
- [STRENGTH] The trust-pillar canonical sentences are pinned verbatim with an
  explicit carve-out from the lone-`spec` rule — `platform.md:562`.
- [STRENGTH] The voice definition is actionable — "'We curate teams' — not 'I
  find people'" — `web.md:242`.
- [GAP] The forbidden-token rule is unenforced — "ESLint rule … evaluated
  post-launch" — `platform.md:564`.
- [GAP] `home/blocks.md` is 1841 lines of layered amendment notes; the
  ForFounders contract is reachable only via a delta section —
  `blocks.md:1623-1636`.
- [GAP] Copy contracts rarely give a character budget — only `blocks.md:1257`
  states a length intent.

**Verdict:** The voice is well-defined and the cold-pitch discipline is the
strongest single content artefact; EN copy is native-grade. Held below 8 by the
unenforced forbidden-token rule and `blocks.md` amendment-sprawl. Highest-
leverage fix: ship the ESLint forbidden-token gate.

#### Native Russian copywriter + editor — 4/10

- [GAP] The RU manifesto — the canonical source per its own front-matter — is an
  explicit placeholder — `manifesto.ru.md:9-12`.
- [GAP] Broken provenance — `manifesto.en.md:4` declares `translates:
  manifesto.ru.md`, but RU is a placeholder while EN carries full prose.
- [GAP] Code-switching in chrome — `ru/web.json` `home.hero.subWithFounders`
  reads "Архитектурная диагностика для founder'ов"; "основателей" exists.
- [GAP] Inconsistent localization — `ru/web.json` keeps English "Founders"
  eyebrow while RO localizes to "Fondatori".
- [STRENGTH] Locale parity is mechanically sound — `ru/web.json` carries all 575
  keys, parity-enforced — `i18n.md:168`.
- [GAP] `i18n.md:115` is stale — claims "six files … ~435 lines"; reality is 18
  catalog files, ~770 keys.

**Verdict:** The canonical RU manifesto is unwritten by design, the translation
graph is false, and chrome contains Latin/Cyrillic code-switching no native
editor would pass. Highest-leverage fix: replace "founder'ов" with "основателей"
throughout `ru/web.json` and author the real Pavel-voice manifesto.

#### Native Romanian copywriter + editor — 4/10

- [GAP] False-friend error in chrome — `ro/common.json` `brand.descriptor` and
  `ro/web.json` `home.hero.subWithFounders` use "echipe curate" for "curated
  teams"; in Romanian *curat* means *clean* — correct is "echipe selectate".
- [GAP] Untranslated English in RO body copy — `ro/web.json`
  `home.forFounders.heading` keeps "Architecture diligence" verbatim.
- [GAP] `manifesto.ro.md:5,10` — `translation_status: draft` translating a
  Russian source that is itself a placeholder.
- [GAP] Anglicism inconsistency — `ro/web.json` `home.forFounders.cta.label`
  reads "Vezi trackul fondatorilor".
- [STRENGTH] The RO manifesto prose, where it commits to Romanian, is fluent and
  grammatically clean with correct diacritics — `manifesto.ro.md:16`.
- [STRENGTH] Full catalog parity — all 575 keys — `i18n.md:168`.

**Verdict:** The RO manifesto reads native-grade, but the marketing catalogs are
AI-drafted and unreviewed — "echipe curate" is a textbook false-friend error.
Highest-leverage fix: a native-speaker review pass over `ro/web.json` starting
with "echipe curate" → "echipe selectate".

#### Native Hebrew copywriter — 6/10

- [STRENGTH] The corpus is honest about why Hebrew was pulled — "the AI-drafted
  strings were not worth maintaining ahead of a native-speaker review pass" —
  `platform.md:541`.
- [STRENGTH] The removal is consistently recorded — `i18n.md:71`,
  `add-muse-translation-mode/proposal.md:35`, `design.md:164`.
- [STRENGTH] RTL infrastructure was deliberately preserved —
  `@radix-ui/react-direction` `<DirectionProvider>` stays (`ui.md:281`), the
  RTL-flip test contract survives (`linkable-ui.md:388`).
- [GAP] The re-entry gate is qualitative — "qualified IL intake volume" — with
  no numeric threshold, though `intake.json` carries a `locale` field
  (`i18n.md:289`).
- [GAP] `add-muse-translation-mode` excludes Hebrew from v1 (`design.md:195`
  Q-MTM-4 closed "No") — the tooling meant to produce reviewed translations
  will not cover HE.
- [GAP] `i18n.md:71` asserts `<html dir="ltr">` but gives no spec for a
  per-locale `dir` switch.

**Verdict:** For a locale that does not ship, the corpus handles Hebrew well —
honest removal, RTL plumbing kept, decision propagated. Held at 6 because the
re-entry path is underspecified. Highest-leverage fix: commit a concrete
IL-intake threshold and add HE to a future phase of `add-muse-translation-mode`.

---

## Ranked remediation

Ordered by leverage — effort against trust-impact.

1. **One editorial reconciliation pass on internal contradictions.** Cheapest,
   highest trust-impact. Fix the `design-system.md` dead reference, the Forge
   orchestrated-review shipped/unshipped flip, the three `--text-display-xl`
   definitions, the model-ID drift (`4-6`/`4-7`, verifier/reviewer), the Mirror
   double-definition, the Anthropic-DPA contradiction, the duplicated blocks.
   None of these need a new decision — only consistency.
2. **Author or restore the four missing design-system files** —
   `design-tokens.md`, `design-motion.md`, `design-components.md`,
   `design-system.md`. Until then `ui.md`'s ~30 cross-references are dead links
   and tokens / motion / contrast have no spec home.
3. **Ratify the founder-asserted strategic decisions, or run the validation.**
   Close the stale Q-POS-2 gate; pick one category in a decision row; either
   commission the PSM / Dunford / DevRel streams or explicitly label the floors
   and category as founder-judgment with that label.
4. **Ship the enforcement gates** — forbidden-token ESLint rule, webhook-HMAC
   verification contract, `createAdminClient` service-role guard (pick one of
   the three options the spec already names).
5. **Native-speaker review pass on RU/RO catalogs.** The cheapest credibility
   fix — "echipe curate" and "founder'ов" are on the live marketing surface.
6. **Legal close-out** — draft the SOW/engagement-contract template, add a
   LICENSE file + trademark-reservation notice to the public mirror, and add an
   Israeli privacy counsel to the D-LB-7 review gate.

---

*Audit produced 2026-05-22. Twenty-five AI-persona specialists, repo-direct
read, every finding grounded at `file:line` per `audit-discipline.md`. The
panels are AI personas — per `specialist-network.md`, escalation to live
specialists is the audit-discipline-gated next step for any finding that becomes
a Linear issue.*
