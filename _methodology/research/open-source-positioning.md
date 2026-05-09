# Research thread — open-source positioning for Rapoport Studio

> **Status:** open research, fact-gathering pass with strategic synthesis. Source-cited.
> **Owner:** Pavel (founder).
> **Opened:** 2026-05-07.
> **Feeds into:** `studio-charter.md` (positioning / philosophy section), `specialist-network.md § 0` (commercial thesis), `value-attribution.md` (Q-VA-future — naming new professions; see Open loops below), `authorship-as-creator-identity.md` (legitimacy layer; not yet in repo), `legal-landscape-authorship.md` (license-choice constraints; not yet in repo).
> **Path note:** placed under `_methodology/research/` per the corrected convention surfaced in the prior critique. Sister files (`value-attribution.md`, `openspec-block-authorship.md`) live alongside.

---

## Why this thread exists

Pavel asked: *what companies operate with a stated public philosophy, what philosophies exist in open-source, what could the studio open-source, and can specs themselves be released as open-source starting points for new projects?*

These four questions are connected by one substrate: **what the studio chooses to make public is itself an act of positioning**. The choice is not "open vs closed" — it is which layer to open, under which license, and what theory of value it expresses. Rapoport Studio operates on top of an open-source-adjacent ecosystem (OpenSpec, Anthropic SDK, Cloudflare, Supabase) but has not yet decided what it itself contributes back to that ecosystem and on what terms.

This thread maps the landscape and ends with a concrete recommendation, not a mechanism spec.

---

## 1. Two traditions of "philosophy as positioning"

### 1.1 Manifesto-driven companies

A small set of companies are known publicly less for their products than for the ideological framework around their products. Three patterns hold across them:

**37signals (Basecamp, HEY, ONCE).** Founded 1999 by Jason Fried & David Heinemeier Hansson. Started with a public manifesto — "37 nuggets of online philosophy and design wisdom" at 37signals.com — *before* the first product. Manifesto preceded Basecamp by five years. Continued public output: REWORK (2010), REMOTE (2013), Shape Up (2019, freely readable online), the public 37signals Employee Handbook on GitHub (basecamp/handbook), public guides to internal communication, public guides to writing, the entire 37signals.com site as a "catalog of ideas" with 81 numbered signals.

The pattern: **the philosophy *is* the marketing**. They never ran traditional advertising. Audience accumulated around the manifesto; product launches landed into a pre-built audience.

**GitLab.** A 13,800-page public handbook (~handbook.gitlab.com), readable by anyone, used as both onboarding for employees across 65+ countries and a recruiting filter that pre-aligns candidates. GitLab's "Transparency" value is operationalized: defaults are public, restricted is the exception with stated reason.

The pattern: **transparency itself is the product feature**. Customers buying GitLab the tool are buying into the operating model that made GitLab the company — the handbook *demonstrates* the methodology rather than describing it.

**Patagonia.** "We're in business to save our home planet" (2018 mission rewrite). Activism budget, public environmental impact reports, 1% for the Planet, Worn Wear repair program, Common Threads pledge. In 2022 transferred company ownership to a trust + nonprofit aligned with mission.

The pattern: **mission as constraint on business decisions**. The philosophy doesn't just market the products — it determines which products get made and which customers are turned away.

### 1.2 What's actually shared across these examples

- **Philosophy precedes or co-founds the product**, never bolted on after revenue.
- **The philosophy is published in operational form** — a handbook, a manifesto, a mission rewrite — not aspirational copy on an "About" page.
- **Adherence is demonstrated through cost** — Patagonia turned down customers; 37signals declined VC; GitLab keeps the handbook public despite the recruiting headache it creates.
- **The audience accumulates around the philosophy**, then converts to customers — not the reverse.

Source: 37signals.com; signalvnoise.com archives; basecamp/handbook (GitHub); Microsoft for Developers blog; Patagonia public mission documents.

---

## 2. The open-source license-philosophy spectrum

Not one open-source movement — at least four traditions with distinct values, and a fifth emerging.

### 2.1 Free Software Foundation (FSF) — ethical

Founded 1985 by Richard Stallman. Anchors on **four freedoms**: run, study, modify, redistribute. The ethical claim is that proprietary software is a *social problem* (denial of freedom), and the solution is to migrate users to free software. Copyleft licenses (GPL family) enforce reciprocity: derivatives must remain free.

**Stance:** software freedom as moral imperative. Non-free software is unjust regardless of quality.

### 2.2 Open Source Initiative (OSI) — pragmatic

Founded 1998 by Bruce Perens, Eric Raymond, and others. Coined "open source" to make FSF principles palatable to enterprise. Maintains the **Open Source Definition** (10 criteria) as the gatekeeping standard for what licenses are "officially" open source.

**Stance:** open source as superior development model. Non-free software is an inferior solution, not a moral failure. Stallman has called OSI "a non-movement" because it doesn't campaign.

### 2.3 Source-available — practical fence

Code visible, code modifiable, but commercial use restricted. Examples: Business Source License (BUSL, MariaDB origin), Server Side Public License (SSPL, MongoDB origin), Elastic License v2.

**Stance:** transparency is good, free-riding by AWS-class hyperscalers is the actual problem. Author retains exclusive commercial right.

### 2.4 Fair Source — eventual open source

Newer category formalized 2024 by Sentry. Code is source-available with three commitments: (a) publicly readable, (b) third-party modification and redistribution allowed with minimal restrictions, (c) **Delayed Open Source Publication (DOSP)** — license auto-converts to Apache 2.0 or MIT after a fixed period (FSL: 2 years; BUSL: 4 years).

Sentry's Functional Source License (FSL) is the clearest articulation. Liquibase Community moved to FSL in 2025. GitButler (Scott Chacon, GitHub co-founder) launched on FSL.

**Stance:** "freedom without free-riding." The author gets a 2-4 year economic moat against competing commercial reuse; the community gets eventual full openness. Compliance departments at 10,000+ organizations have approved Sentry's FSL — operationally indistinguishable from open source for internal use.

**Why this matters here:** FSL is the most relevant license-philosophy for a studio that wants to be public-and-collaborative without giving competitors a free fork to undercut the studio's own services.

### 2.5 Ethical source — values-conditioned

Newest and most contested. Hippocratic License (Coraline Ada Ehmke, 2020), Anti-996 License, others. License grants conditioned on the licensee not violating UN human rights principles / not exceeding 996 work hours / etc.

**Stance:** software freedom is contingent on user behavior. If you abuse human rights, you don't get to use the code.

OSI has rejected most ethical-source licenses as not meeting Open Source Definition criterion 6 ("no discrimination against fields of endeavor"). Bruce Perens has criticized them as unenforceable and self-contradictory.

### 2.6 Creative Commons — adjacent

Not for software (mostly) but for content: text, design, methodology, documentation. CC-BY (attribution), CC-BY-SA (attribution + share-alike), CC-BY-NC (non-commercial), CC0 (public domain dedication). This matters for **methodology and templates** — what the studio writes that isn't code.

CRediT taxonomy itself is CC-BY licensed (per `legal-landscape-authorship.md § 9`). Wikipedia content is CC-BY-SA. Most "open philosophy" content (handbooks, manifestos) chooses CC-BY or CC-BY-SA.

### 2.7 What the studio's context narrows to

Given the studio's IP model B (mutual license; client owns deliverable, studio retains methodology), Moldovan inalienable moral rights, and the absence of competitor pressure at this scale, the realistic license universe is:

| Layer | Realistic license options |
|---|---|
| Methodology docs (charter, network, philosophy) | CC-BY 4.0 or CC-BY-SA 4.0 |
| OpenSpec capability templates | MIT or Apache 2.0 |
| Forge CLI engine | MIT, Apache 2.0, or FSL-1.1-Apache-2.0 |
| Muse runtime / prompt augmentations | MIT, FSL, or proprietary |
| Mirror schema | MIT or CC-BY |
| Per-engagement OpenSpec corpus (delivered to client) | client-owned per IP-B; license is between studio and client, not public |

---

## 3. Spec-driven development — the active landscape

This is where the studio operates *today*, whether positioned publicly or not. By early 2026, four spec-driven development tools dominate, with combined ~137,000 GitHub stars.

### 3.1 GitHub Spec Kit

- Maintainer: GitHub (Microsoft).
- Stars: ~75,000.
- License: MIT.
- CLI: `specify` (Python, uv-managed).
- Approach: heavyweight, phase-gated. A **constitution** document encodes project standards; every feature flows through `/specify` → `/clarify` → `/plan` → `/tasks` → `/implement` → `/analyze`.
- AI agent support: 18+ (Copilot, Claude Code, Cursor, Gemini CLI, Cline, Roo Code, others).
- Strength: depth, audit trail, governance.
- Weakness: framework metadata consumes significant context window; setup is heavier; phase enforcement creates friction.

### 3.2 Fission-AI OpenSpec

- Maintainer: Fission-AI / @0xTab.
- Stars: ~25,000–28,000.
- License: MIT.
- CLI: `openspec` (Node.js).
- Approach: lightweight, brownfield-first. Each change gets a folder (`openspec/changes/<slug>/`) with proposal / specs / design / tasks. Source of truth at `openspec/specs/`. 50KB context limit *enforced*.
- AI agent support: 20+ via slash commands and AGENTS.md convention.
- Strength: speed, fluidity, zero ceremony, brownfield support, minimal context cost.
- Weakness: less governance scaffolding than Spec Kit; less suited to large compliance-heavy orgs.

**This is what Rapoport Studio uses.** The `openspec/changes/<slug>/proposal.md`, `design.md`, `tasks.md` triplet structure throughout the studio's monorepo is the OpenSpec-CLI structure verbatim.

### 3.3 GSD (Getting Stuff Done)

Smaller, execution-oriented. Less specification-pure, more about orchestrating multi-step AI work.

### 3.4 Taskmaster AI

Task-orchestration focus rather than specification depth. Subscription product.

### 3.5 What this means for the studio's positioning

**Critical observation:** the studio is *already* downstream of an open-source standard (Fission OpenSpec) without having declared its position relative to it. Three possible postures:

- **Silent consumer** — use OpenSpec, never reference it in public materials. Status quo. Reputation costs nothing, contributes nothing back.
- **Acknowledged consumer + occasional contributor** — credit OpenSpec in studio materials, contribute upstream when bugs surface. Low-cost, honest.
- **Methodology-layer publisher** — release the studio's own contributions *on top of* OpenSpec as a separate public artifact. This is the strategic position: not competing with OpenSpec the CLI, but extending what OpenSpec corpora *do* with social architecture (specialist network, contribution model, IP-B engagement charter, philosophy framework).

The third posture is what "studio open-sources methodology" actually means in practice. Spec Kit and OpenSpec already own the CLI/tooling layer. The studio's distinctive contribution is the *guild and contract layer* on top.

---

## 4. What the studio could open-source — five layers from cheapest to most ambitious

Ordered by cost-to-ship and ranked against strategic value.

### Layer 1 — Charter and methodology documents (CC-BY)

**What:** `studio-charter.md`, `specialist-network.md`, `rapoport-studio-engagement.md`, the upcoming `studio-philosophy.md`, the four research files.

**Cost:** zero. They already exist.

**Value:**
- Demonstrates the methodology in operational form (the GitLab pattern).
- Builds an audience pre-product (the 37signals pattern).
- Gives prospective clients a way to read the studio before signing.
- Converts the legal-floor moral-rights obligation into a brand asset (`legal-landscape-authorship.md § 10` — the studio is *required* to honor authorship; making this public is also marketing).

**Risk:** competitors copy the methodology. **Counter:** the methodology is documented; the *execution* is not. Copying a charter does not buy a network.

**License recommendation:** CC-BY 4.0 — anyone can use, modify, redistribute with attribution. Studio retains authorship credit. Inalienable Moldovan moral rights are honored automatically.

### Layer 2 — Capability templates (MIT)

**What:** the `_templates/capability.md` shape, the change-folder triplet schema (proposal/design/tasks), the frontmatter conventions, the dependency-graph patterns.

**Cost:** low. Extract from existing repo, normalize.

**Value:**
- Other studios can adopt the patterns directly.
- Establishes the studio as a contributor to the spec-driven dev movement, not just a consumer.
- Compatible with both OpenSpec CLI and Spec Kit CLI as input — the templates sit *above* the tooling layer.

**Risk:** templates without the methodology context produce shallow imitations. **Counter:** the methodology docs in Layer 1 explain *why* the templates are shaped this way.

**License recommendation:** MIT — maximum permissive, OSI-approved, suits templates that should propagate freely.

### Layer 3 — Forge CLI engine (FSL or Apache 2.0)

**What:** `packages/forge` — audit, spec, review, estimate commands. The engine that runs against an OpenSpec corpus and produces structured outputs.

**Cost:** medium. Forge is currently `experimental`, has VIVOD-specific MODULE_REGISTRY blockers (AI-52/AI-53 per memory). Neutralization is required before public release.

**Value:**
- Tools propagate faster than methodology. Forge is the bridge that lets a non-studio team experience the methodology without engaging the studio.
- Establishes the studio's technical credibility in the SDD landscape.

**Risk:** competitors fork Forge and build a competing studio. **Counter:** this is exactly what FSL is designed for. 2-year exclusivity period protects the studio's commercial position; after 2 years, code converts to Apache 2.0 and ecosystem benefits.

**License recommendation:** FSL-1.1-Apache-2.0. This is Sentry's choice and has been validated at scale — Liquibase, GitButler, others have followed. Pavel's memory of Sentry's $100M revenue under FSL is the existence proof that Fair Source supports a real services business.

### Layer 4 — Mirror schema (MIT)

**What:** the perceptual-spec layer schema described in `mirror.md` — `client_vocabulary`, `key_screens`, `brand_anchors`, `forbidden_patterns`, `comparable_products`, `out_of_scope`, plus the prose sections.

**Cost:** low-medium. Schema is stable enough; render pipeline is studio-specific and stays proprietary.

**Value:**
- Mirror is genuinely novel — perceptual-spec-as-a-deliverable doesn't exist elsewhere yet. Publishing the schema seeds a category.
- Other studios doing client work get a vocabulary for the *non-technical* spec layer that doesn't exist in OpenAPI or AsyncAPI.

**Risk:** schema published without the social architecture (`muse.md` write_mirror_field, `forge mirror-export`, the IP model B export) gets misused. **Counter:** publish schema + reference the methodology layer (Layer 1) so adopters can find the operating context.

**License recommendation:** MIT for the schema definition; CC-BY for any accompanying methodology docs.

### Layer 5 — Specialist contribution model and contributor agreement template

**What:** the `specialist-network.md` certification ladder, the `D-NW-*` decision pattern, the contribution-tier vocabulary, a Moldovan-law-grounded contributor template (output of `Q-LL-1` from `legal-landscape-authorship.md`).

**Cost:** medium-high. Q-LL-1 is not yet drafted; needs counsel sign-off (Andrei at i-avocat.md) before public release.

**Value:**
- This is the *most* distinctive thing the studio does. SDD tools are commodity by 2026. Specialist networks operating under explicit IP terms with named contribution tiers and inalienable-moral-rights-aware contracts are *not* commodity.
- A Moldovan-law-grounded master contract template, once drafted, is reusable infrastructure for any studio operating in the same regime.
- Ties directly to the philosophy thread: "studio names new professions" requires public artifacts that name them.

**Risk:** legal exposure if a third party adopts the contract template incorrectly. **Counter:** publish with explicit "not legal advice; consult counsel" disclaimer (the legal-landscape file already does this); structure as a *template* that requires per-engagement adjustment.

**License recommendation:** CC-BY-SA 4.0 — share-alike requires that derivative contracts also be public and attributable, growing a commons of public-contract templates rather than letting the studio's work get privately enclosed.

---

## 5. Specs as starting points — Pavel's specific question

Pavel's framing: *"спеки можно хранить прямо в формате open source и предлагать начинать проекты."*

This is a real and concrete possibility. Two prior arts already do versions of it:

### 5.1 Spec Kit's "constitution" + templates

GitHub's Spec Kit ships **opinionated templates** that bootstrap a new project's spec discipline. The constitution is a project-scoped file that encodes standards; the spec template defines the proposal shape; the plan template defines the technical approach.

A new team running `specify init` gets a working SDD environment in minutes, with templates that have been refined across thousands of users.

### 5.2 OpenSpec's `openspec init`

Same pattern, lighter. `openspec init` scaffolds the directory structure, the AGENTS.md handoff, the slash-command bindings for the chosen AI agent.

### 5.3 What Rapoport Studio could add — Mirror-aware project starters

Neither Spec Kit nor OpenSpec currently ship **starter spec corpora** — opinionated initial corpora for specific project shapes (e.g., "client-portal SaaS," "regulated-industry SaaS with GDPR baseline," "AI-orchestration product," "mental-health adjacent product with clinical-data sensitivity").

The studio's own corpus already encodes one of these shapes (`platform.md`, `muse.md`, `mirror.md`, `forge.md`, `studio.md`, `web.md`, etc.). With anonymization and generalization, this becomes a **public starter** — `rapoport-studio/openspec-starter-ai-orchestration` — that any team can clone as the seed of their own corpus.

This is **the studio's most distinctive open-source possibility** that doesn't yet exist anywhere. Specifically:

- Spec Kit ships templates per *artifact* (one constitution, one spec, one plan).
- OpenSpec ships templates per *change* (one proposal, one design, one tasks).
- Nothing currently ships templates per *system shape* — a coherent multi-capability starter corpus.

The studio is uniquely positioned because:
1. It already maintains 14+ capability files in production.
2. It has an opinion about what shape they should take (the methodology layer).
3. Its IP-B model already separates "deliverable" from "methodology" — the methodology layer is already partitioned for export.

A v1 starter corpus might ship as:

```
rapoport-studio/openspec-starter-ai-orchestration/
├── README.md                          # what this is, license, attribution
├── LICENSE                            # CC-BY-SA 4.0 (corpus) or MIT
├── openspec/
│   ├── current/
│   │   ├── platform.md                # generic AI-orchestration platform skeleton
│   │   ├── auth.md                    # Supabase + RLS pattern
│   │   ├── canvas.md                  # AI-conversation surface skeleton
│   │   ├── muse.md                    # GPAI-orchestration skeleton
│   │   ├── mirror.md                  # perceptual-spec-layer schema
│   │   ├── forge.md                   # CLI engine spec
│   │   ├── compliance-and-safety.md   # LINDDUN + DPIA hooks
│   │   ├── data-privacy.md            # GDPR/MD-195/2024 baseline
│   │   └── design-system.md           # token + component skeleton
│   ├── _methodology/
│   │   ├── charter.md                 # CC-BY-SA derivative of studio charter
│   │   ├── network.md                 # how to think about specialists
│   │   ├── philosophy.md              # naming/propagation/legitimacy frame
│   │   └── research/
│   │       └── PROCESS.md             # 5-step research-to-spec discipline
│   ├── _templates/
│   │   ├── capability.md              # 11-section template
│   │   ├── change/proposal.md
│   │   ├── change/design.md
│   │   └── change/tasks.md
│   ├── decisions.md                   # initial D-rows for the starter shape
│   ├── dependencies.md                # initial capability dependency graph
│   ├── maturity.md                    # initial scorecard (all stubs at maturity 1)
│   ├── open-questions.md
│   └── AGENTS.md                      # OpenSpec-compatible agent handoff
```

A team running `git clone` + `openspec init --skip-scaffolding` against this gets ~100 hours of methodology work as their starting point. The studio's name is on every file (CC-BY-SA attribution); the studio's philosophy is the foundational frame; the team can fork freely.

This is genuinely civilizational work — it lowers the barrier for any team to operate at the studio's methodology standard without needing to hire the studio. It's also *commercial* — teams that grow past the starter inevitably hit problems where studio engagement becomes cheaper than continued solo work.

---

## 6. Risks — honest list

### 6.1 Free-riding (Layers 3–5)

A competitor clones the starter, the Forge engine, and the contribution model, and offers a competing studio service. **Mitigated by** FSL (2-year exclusivity on Forge), CC-BY-SA (share-alike on contribution templates), and the network itself (which is the actually unforkable asset).

### 6.2 Reputation pressure to maintain

Once published, broken examples or stale specs in the public corpus reflect on the studio. **Mitigated by** explicit `Status: Active | Stub | Deprecated` markers per file (already part of studio's capability template); maturity scorecard published alongside (already part of `_methodology/`).

### 6.3 Premature canonicalization

Publishing the methodology before it's stable risks fixing patterns that will look wrong in a year. **Mitigated by** explicit Phase-1-research / Phase-2-closing convention (already in studio's discipline); semantic versioning of the public corpus.

### 6.4 Confusion with existing OpenSpec CLI

Users may expect "Rapoport OpenSpec" to be a CLI. **Mitigated by** explicit framing: "Rapoport Studio Methodology Corpus, runs on Fission-AI/OpenSpec or GitHub Spec Kit" — name disambiguation matters.

### 6.5 Competitor frames "Rapoport methodology" as overhead

Cheaper studios can market "we don't burden you with paperwork." **Mitigated by** outcome data (when the studio has it) — engagements run on the methodology vs without. Currently the studio has zero datapoints; this risk is real until first 3 engagements complete.

### 6.6 Legal exposure on contract templates (Layer 5)

If a third party adopts a Moldovan-law-grounded contract template incorrectly. **Mitigated by** explicit disclaimer; counsel review (Q-LL-1) before any public release; structuring as a *template* requiring jurisdictional adjustment.

---

## 7. Recommendation

Three-phase rollout, sequenced by cost and risk:

### Phase 0 (now, no cost) — narrative posture

Add to `studio-charter.md` and the public web copy a single sentence: **"Rapoport Studio is built on the open-source ecosystem and contributes back to it."** Stop using methodology as a private competitive advantage in public-facing copy. Acknowledge OpenSpec, Spec Kit, Anthropic SDK, Cloudflare, Supabase as upstream dependencies the studio relies on.

This commits the studio to the trajectory without yet shipping anything.

### Phase 1 (3–6 months) — Layers 1 + 2 published

Public repo `rapoport-studio/methodology` containing:
- `studio-charter.md` (CC-BY 4.0)
- `specialist-network.md` (CC-BY 4.0)
- `studio-philosophy.md` (when written; CC-BY 4.0)
- `_templates/capability.md` and change-folder templates (MIT)
- The four `_methodology/research/` files (CC-BY 4.0)

**RAP-119-conditional candidates (added 2026-05-07):**
- `loop-protocol.md` (gated on RAP-119 Phase 8.4 — corpus-promotion decision after Sessions 1–4 validate the rhythm). If 4-of-5 validation criteria pass, this becomes a Layer 1 candidate. Now lives at [`openspec/_methodology/research/loop-protocol.md`](./loop-protocol.md) (research-level register).
- `team-onboarding.md` (gated on RAP-119 D7 — Pavel writes the meta-process triplet inside the team's corpus by Session 3). Reflexive document: methodology that documents itself by use. Layer 1 candidate post-Session 4 if the artifact ends up engagement-clean.

> **Update 2026-05-08 — RAP-119 cancelled.** Sessions 1–4 never ran;
> Phase 8.4 corpus-promotion gate for `loop-protocol.md` is reopened
> pending alternative empirical use (the protocol survives at
> [`./loop-protocol.md`](./loop-protocol.md), validation-pending). The
> `team-onboarding.md` Layer-1 candidacy is **withdrawn** — D7 was
> never produced and reanimating a never-deployed exemplar would be
> dishonest. First real multi-author engagement that runs becomes the
> next gate.

**Sessions 1–4 of RAP-119 were intended as a natural gate before Phase 1 publish.** Risk 6.5 ("competitor frames methodology as overhead — mitigated by outcome data") was honest: pre-RAP-119 the studio had zero datapoints. With RAP-119 cancelled the studio still has zero non-Pavel engagement datapoints, so Phase 1 publish remains gated on alternative empirical sources. Empirical wiring rationale lives in [`rap-119-as-research-bed.md`](./rap-119-as-research-bed.md) § 3.3 (preserved as how-to-instrument template).

This costs roughly one week of editing for redaction (engagement-specific names → generic placeholders) plus a license decision.

### Phase 2 (6–12 months, depends on Phase 1 reception) — Layer 5 starter corpus

If Phase 1 attracts inbound interest (forks, citations, prospect questions), invest in Layer 5: the AI-orchestration starter corpus. This is the highest-value and highest-cost deliverable.

Gating decision happens once the studio has its first 3 engagements archived — the starter corpus needs to encode patterns observed across 3+ real engagements, not be theoretical.

### Phase 3 (deferred, 12+ months) — Forge engine FSL release

Only after Forge is neutralized of VIVOD-specific dependencies (AI-52 / AI-53) and has been used in at least 2 non-VIVOD engagements. Public release on FSL-1.1-Apache-2.0.

This may never happen — the studio may correctly conclude that Forge is too thin to be a standalone public artifact. That's a valid outcome.

---

## 8. Open questions

| ID | Priority | Question |
|---|---|---|
| `Q-OS-1` | P0 | Does the studio commit to the narrative posture (Phase 0)? This is the gateway to everything else and costs nothing to commit to. |
| `Q-OS-2` | P0 | License choice for methodology docs: CC-BY 4.0 (anyone uses with attribution) or CC-BY-SA 4.0 (anyone uses with attribution + share-alike on derivatives)? Recommendation: **CC-BY-SA** for charter + network + philosophy; **CC-BY** for research threads and individual templates. |
| `Q-OS-3` | P1 | Public repo name: `rapoport-studio/methodology` or `rapoport-studio/openspec` (collides with Fission-AI namespace) or `rapoport-studio/handbook` (37signals echo) or other? Recommendation: `methodology` — descriptive, distinct, non-colliding. |
| `Q-OS-4` | P1 | Whether to release the four research files as part of Phase 1 or wait until they reach `Status: superseded`. Recommendation: release with `Status: open research` clearly marked. The research-in-public posture is itself a positioning choice (the GitLab handbook precedent — incomplete + public > complete + private). |
| `Q-OS-5` | P2 | Layer 5 starter corpus name and scope — `openspec-starter-ai-orchestration` is descriptive but long. Could be `rapoport-platform-starter` or `studio-starter`. Defer until Phase 2. |
| `Q-OS-6` | P2 | Whether to file the studio's own contributions back to Fission-AI/OpenSpec upstream (e.g., capability template improvements, AGENTS.md conventions). Low cost, high reputation value. |

---

## 9. What this thread does NOT do

- It is **not** a license decision. Recommendations above are starting points; counsel and Pavel make the final call.
- It is **not** a marketing plan. Positioning is upstream of marketing; this thread sets the position, not the campaign.
- It is **not** a commitment to ship. Phase 0 commits to the narrative posture; Phases 1–3 are conditional.
- It is **not** the Q-VA-future mechanism for `value-attribution.md`. "Naming new professions" requires public artifacts (charter pages); this thread provides the *vehicle* (open-source publication) but not the *content* (which professions get charters). See Open loops below — the Q5 referenced in earlier drafts of this file does not currently exist in `value-attribution.md`; closest existing anchor is Q2 (pricing of new professions), which is downstream of naming.

---

## 10. How this thread closes

Either:

1. Pavel commits to Phase 0 (narrative posture). Phase 1 gets scheduled. This file becomes a stable reference until Phase 1 ships, at which point it's amended with what shipped and what got cut.
2. Pavel concludes the studio is not ready for public positioning yet. The file becomes a watch-item with a re-open trigger (e.g., "after first non-Pavel engagement archives").
3. Pavel concludes the open-source posture is wrong for the studio (e.g., wants to remain fully proprietary methodology). The file is closed as resolved-negative; the conclusion is recorded as a `D-*` row.

Any of the three is valid. The thread does not advocate for option 1 over 2 or 3 — it provides the substrate for the decision.

---

## 11. Adjacent prior thinking

### 11.1 Existing in repo (links resolve today)

| Source | Relationship |
|---|---|
| `studio-charter.md § 1` ("AI as orchestration substrate") | This thread proposes making that thesis public-facing through methodology release. |
| `specialist-network.md § 0` (commercial thesis) | Layer 5 (contributor model + contract template) is the operational corollary. |
| `value-attribution.md` Q2 (how new professions are valued before market data exists) | Closest existing anchor. This thread provides the publication vehicle (open-source charter pages); Q2 is the pricing dimension. The naming dimension referenced as "Q5" in earlier drafts of this file is not yet in `value-attribution.md` — see Open loops § 11.3. |
| `mirror.md` | Schema is a Layer 4 candidate for public release. |
| `forge.md` | Engine is a Layer 3 candidate, gated on de-VIVOD-ification. |
| `openspec-block-authorship.md` | Sister research thread under `value-attribution.md` Q1. |
| [`competitive-positioning-sdd-frameworks.md`](./competitive-positioning-sdd-frameworks.md) | Sister positioning thread. This file decides *what* to publish; competitive-positioning decides *what to say* relative to Spec Kit, OpenSpec, Kiro, BMAD, Thoughtworks, 37signals. Phase 1 of this file is one of two gates that close that one. |
| [`rap-119-as-research-bed.md`](./rap-119-as-research-bed.md) | Observation protocol that wires Q-OS-1 (narrative posture commitment) and Phase 1 Layer 1 candidate readiness to RAP-119 surfaces. Sessions 1–4 are the natural gate before Phase 1 publish. |

### 11.2 Forward references — files cited but not yet drafted

This thread references three sister files that do not yet exist in the repo. References are kept intact so that if/when those files are integrated, the cross-links resolve. Until then, treat the citations as forward-referenced placeholders.

| Cited file | Where referenced | Current status |
|---|---|---|
| `authorship-as-creator-identity.md` | file-header (`Feeds into`); § 4 Layer 1 (philosophy thread); § 11 (legitimacy layer) | Drafted in chat 2026-05-07 with five integration questions outstanding; not yet placed in repo. |
| `legal-landscape-authorship.md` | file-header; § 2.6 (CRediT licensing); § 2.7 (IP model B + Moldovan moral rights ground); § 4 Layer 1, Layer 5 (Q-LL-1 gating); § 9; § 10 Phase 2; § 11 | Not yet drafted. The license recommendations in § 2.7 and § 4 rest on this file's claims about IP model B and Moldovan inalienable moral rights — those claims have not been counsel-reviewed. |
| `studio-philosophy.md` | file-header; § 4 Layer 1 (lists as current methodology); § 7 Phase 1 (Phase 1 deliverable) | Not yet drafted. Acknowledged in this file as "upcoming". |

### 11.3 Open loops — concepts referenced as if they exist

| Loop | Resolution path |
|---|---|
| `value-attribution.md` Q5 ("naming new professions") | Q5 does not currently exist in `value-attribution.md`; only Q1–Q4. This file's "Q-VA-future" placeholder marks the gap. Resolution: either add Q5 to `value-attribution.md`, or accept that "naming" is upstream of Q2 (pricing) and cite Q2 directly. |
| Starter corpus `decisions.md` / `dependencies.md` / `maturity.md` / `open-questions.md` (§ 5.3) | These files are part of the **proposed** starter corpus shape, not the studio's actual corpus. The studio's `openspec/` does not currently include them at root. § 5.3 is therefore a hypothetical artifact, not an export of existing structure. |
| Sentry $100M revenue under FSL claim (§ 4 Layer 3) | Cited as "Pavel's memory" — not independently verified in this thread. Verify before any public Phase 1 publication. |
| Sources in § 12 | Several citations (arxiv 2602.00180, Hightower Feb 2026, Big Hat Group Mar 2026, Microsoft Spec Kit blog Sep 2025) are future-dated relative to typical training cutoffs and may need human verification before any Phase 1 publication. The 37signals / GitLab / Sentry / Fission OpenSpec / GitHub Spec Kit citations are independently verifiable today. |

---

## 12. Sources cited

| Source | Reference |
|---|---|
| 37signals manifesto, 1999 | 1999.37signals.com/manifesto |
| 37signals current "signals" catalog | 37signals.com (81 numbered signals) |
| Basecamp Employee Handbook | basecamp.com/handbook + basecamp/handbook on GitHub |
| 37signals Guide to Internal Communication | 37signals.com/how-we-communicate |
| GitLab Handbook | handbook.gitlab.com |
| FSF Four Freedoms | gnu.org/philosophy/free-sw |
| OSI Open Source Definition (10 criteria) | opensource.org/osd |
| Stallman, "Why Open Source Misses the Point" | gnu.org/philosophy/open-source-misses-the-point |
| Sentry FSL announcement | blog.sentry.io/introducing-the-functional-source-license |
| Sentry "Now Fair Source" announcement | blog.sentry.io/sentry-is-now-fair-source |
| FSL spec | fsl.software |
| Functional Source License repository | github.com/getsentry/fsl.software |
| Liquibase Community → FSL move (2025) | liquibase.com/blog/liquibase-community-for-the-future-fsl |
| Fair Source movement (TechCrunch) | techcrunch.com (Sept 2024) |
| Hippocratic License (ethical source) | firstdonoharm.dev |
| Creative Commons | creativecommons.org/licenses |
| GitHub Spec Kit announcement | github.blog (September 2025) |
| GitHub Spec Kit repository | github.com/github/spec-kit |
| Fission-AI OpenSpec repository | github.com/Fission-AI/OpenSpec |
| OpenSpec.pro | openspec.pro |
| "Spec-Driven Development: From Code to Contract" (arXiv) | arxiv.org/html/2602.00180v1 |
| Hightower, "GSD vs Spec Kit vs OpenSpec vs Taskmaster AI" | medium.com/@richardhightower (Feb 2026) |
| Microsoft for Developers blog on Spec Kit | developer.microsoft.com/blog/spec-driven-development-spec-kit |
| Big Hat Group, "OpenSpec vs Spec Kit" | bighatgroup.com (March 2026) |
| OpenAPI Specification | swagger.io/specification |
| AsyncAPI Initiative | asyncapi.com |
| Patagonia mission statement, 2018 | patagonia.com (public) |

---

## 13. Change log

| Date | Author | Change |
|---|---|---|
| 2026-05-07 | Pavel (founder) | Opened thread. Five-source-cluster pass: manifesto-driven companies, OSS license-philosophy spectrum, spec-driven development landscape, layer analysis, three-phase recommendation. Six open questions Q-OS-1 through Q-OS-6, P0 on narrative posture and license choice. AI assistance: drafted with Claude (Anthropic). |
| 2026-05-07 | Integration pass | Placed at `_methodology/research/open-source-positioning.md`. Forward-references to three sister files (`authorship-as-creator-identity.md`, `legal-landscape-authorship.md`, `studio-philosophy.md`) retained intact and surfaced in § 11.2. The `value-attribution.md` Q5 reference reframed to Q-VA-future placeholder + Q2 anchor; gap surfaced in § 11.3. Sources flagged for human verification before Phase 1 publication. |
