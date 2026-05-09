# Research thread — competitive positioning vs SDD frameworks and AI-consulting

> **Status:** open research, fact-gathering pass with positioning synthesis. Source-cited with direct user quotes where available.
> **Owner:** Pavel (founder).
> **Opened:** 2026-05-07.
> **Companion to:** [`openspec/archive/2026-05-08-add-founder-track/research/israeli-startup-ecosystem.md`](../../archive/2026-05-08-add-founder-track/research/israeli-startup-ecosystem.md) — Israeli founder ecosystem (different scope; this thread is global SDD-framework + consultancy comparison). Note: companion file currently lives inside the archived `add-founder-track` change-folder, not in `_methodology/research/`; promotion to methodology level is a separate decision.
> **Feeds into:** `studio-charter.md` (positioning copy), [`open-source-positioning.md`](./open-source-positioning.md) (decides what to publish), web copy at `rapoport.studio/services` and `/methodology`.
> **Path note:** placed under `_methodology/research/` per the corrected convention.

---

## 0. What this thread is and is not

Pavel asked: *why is it better to work with us than with other OpenSpec-style frameworks; what do other studios actually do; what do users write about them; what do they love; what problems do they have; and how do we solve those problems specifically?*

This thread answers all six in one document. It is:

- A **competitive positioning analysis**, not an OpenSpec proposal.
- Grounded in **direct user quotes** where available (GitHub Discussions, Hacker News, Reddit, blog reviews) — not vendor marketing.
- Honest about **where we are weaker**, not just where we are stronger.

It is not:

- A pricing comparison (different file).
- An Israeli-ecosystem update (different file — already exists).
- A go-to-market plan (positioning is upstream of marketing).

---

## 1. The competitive landscape at a glance

By Q1 2026, six tools and three studio-categories define this space. Combined GitHub stars across the SDD tools cross 137,000.

| Category | Player | What it is | Stars / scale | License |
|---|---|---|---|---|
| **SDD CLI / IDE** | GitHub Spec Kit | spec → plan → tasks toolkit | ~75,000 | MIT |
| **SDD CLI / IDE** | Fission-AI OpenSpec | brownfield-first change-folder discipline | ~25,000–28,000 | MIT |
| **SDD CLI / IDE** | AWS Kiro | AI-native IDE on AWS, spec-first | early access, no public count | proprietary |
| **SDD CLI / IDE** | BMAD-Method | full-lifecycle multi-agent framework | ~10,000+ | MIT |
| **SDD platform** | Augment Code Intent | living-specs platform with multi-agent orchestration | proprietary, paid | commercial |
| **Adjacent** | Cursor / Claude Code / Antigravity | AI-IDE without spec layer | varies | mixed |
| **Methodology consultancy (large)** | Thoughtworks | global tech-consultancy with AI/works platform | 10,000+ employees, 47 offices | services |
| **Methodology consultancy (boutique)** | 37signals / Basecamp | publishes Shape Up, REWORK, handbook | 30+ years, public manifesto | services + product |
| **AI agency (build-shop)** | ShowcaseIT / AI Superior / Webeet / etc. | AI integration build shops | 50+ named on Verox lists | services |

**Where Rapoport Studio sits:** in the gap between the SDD tools (which are CLIs) and the methodology consultancies (which are services). The studio uses OpenSpec the CLI, contributes the *guild/contract layer* the CLIs don't have, and offers it as a service that produces an artifact the client owns.

This is a real gap. None of the six tools above offers a service. None of the three consultancy categories above offers a portable methodology corpus the client can keep.

---

## 2. Tool-by-tool — what users love, what users hate, what we do differently

### 2.1 GitHub Spec Kit (~75k stars, MIT, sept 2025)

**What people love.**

- Agent-agnostic: works with 18+ AI agents (Copilot, Claude Code, Cursor, Gemini CLI, Cline, Roo Code, others). No vendor lock-in.
- Constitution document encodes project-wide standards once, every spec inherits.
- "If you like the speed of vibe coding but hate the uncertainty," Spec Kit is GitHub's answer.
- Templates flag unknowns as `NEEDS CLARIFICATION` rather than guessing.
- Backed by GitHub/Microsoft, validated by enterprise procurement.

**What people actually complain about (direct quotes from GitHub Discussions, January–March 2026):**

- *"AI forgets everything between sessions. Great specs, but every new chat starts from zero."* (Discussion #1482, Jan 2026)
- *"With the new year coming and Den that left Microsoft to join Anthropic I'm a little worried about how the project is (NOT) progressing. In the last month we had 22 PRs opened and exactly zero commits."* (Discussion #1482) — **maintenance concerns are real**; the original PM left.
- *"After several hours work, I have a..."* (truncated in Discussion #1686 "High Level Design Concerns")
- *"Treating specs as session deltas rather than permanent truth has been the thing that finally removed the friction we kept hitting."* (Discussion #152) — **users are routing around the framework's own model**.
- *"The parts of SDD that really work are the tactical execution docs, which tend to be throwaway for the most part."* (Discussion #152)
- Verbose output: "generates substantial markdown requiring careful human review, creating overhead for simple features" (Augment Code review).
- No multi-agent orchestration — sequential single-agent workflow, no parallel execution.
- Context bloat: framework metadata consumes significant token budget at scale.
- No code review step built in — iteration stops at planning phase.

**What Rapoport Studio does differently:**

| Spec Kit problem | Studio answer |
|---|---|
| AI forgets between sessions | **Persistent context layer.** `_methodology/`, `current/`, `decisions.md`, `archive/`, `_research/` are the project's memory. Every Muse turn reads from disk, not chat history. The studio's `add-decision-provenance-and-outcomes` proposal — even unshipped — captures that this is a known design axis, not an accident. |
| Maintenance dependent on one company | Studio uses OpenSpec **and** is methodology-layer-portable. If Spec Kit dies, our corpus moves to OpenSpec or any successor in a day. We are not betting on one CLI. |
| Verbose output, hard human review | Maturity scorecard + dependency graph + per-file `Status` markers (`Active`/`Stub`/`Deferred`/`Deprecated`) make corpus health auditable at a glance. |
| Specs drift unless manually refreshed | `forge audit` runs spec-fidelity audit against actual code. Drift produces report rows, not silent rot. |
| Throwaway tactical docs vs permanent truth | We separate them structurally. Change-folders are the tactical layer (proposal/design/tasks); `current/` is the durable layer; `archive/` is the historical layer. Users routing around Spec Kit are reinventing OpenSpec's own model — which is what we use. |

### 2.2 Fission-AI OpenSpec (~25–28k stars, MIT, brownfield-first)

**What people love.**

- Lightweight, fluid, no rigid phase gates.
- Brownfield-first — works on existing codebases, not just greenfield (Spec Kit's gap).
- 50KB context limit *enforced* — predictable token costs.
- Change-isolation per folder prevents concurrent-modification chaos.
- Works with 20+ AI assistants via slash commands and AGENTS.md.
- 5-minute setup vs Spec Kit's 30 minutes.
- *"Best fit when modernizing legacy systems — purpose-built for brownfield with its lightweight, change-centric design."* (multiple comparison reviews)

**What people complain about.**

- Less governance scaffolding than Spec Kit. *"Its ceiling is lower but predictable."* (Big Hat Group review)
- *"You can't encode the same depth of governance"* — 50KB context limit is a blunt instrument.
- Less suited to large compliance-heavy orgs.
- Lighter brand — *"backed by Fission-AI / @0xTab"* doesn't carry enterprise weight of GitHub.
- Authorship dispute resolution / contributor attribution is not a built-in concept.

**What Rapoport Studio does differently:**

We don't compete with OpenSpec. **We are downstream consumers of OpenSpec the CLI**, building the methodology layer above it. Specifically:

| OpenSpec gap | Studio answer |
|---|---|
| Less governance / compliance posture | We add `_methodology/` (charter, network, philosophy), `decisions.md` with `D-*` rows, dependency tracking, maturity scorecards. Governance is a corpus convention, not a CLI feature. |
| Light brand for enterprise | We add the **engagement charter** (`rapoport-studio-engagement.md`) with explicit IP model, mode (`deliverables-only` / `hybrid` / `full`), Linear team prefix, comms convention, research contribution policy. Enterprise procurement reads this and approves. |
| No contributor-attribution model | We add `specialist-network.md`, `D-NW-*` rows, contribution-tier vocabulary, and the legal-landscape-grounded contract template (in progress, gating on `Q-LL-1` per `legal-landscape-authorship.md`). |
| No business-model layer | We add IP model B (mutual license), Discovery €5k engagement shape, MITP residency posture, EU AI Act compliance baseline. |

**Honest:** OpenSpec the CLI is good, we use it, and our methodology layer is *complementary*, not competitive. Anyone who likes OpenSpec should keep using OpenSpec — the question is whether they want methodology-layer support around it.

### 2.3 AWS Kiro (early access 2025, IDE, AWS Bedrock)

**What people love.**

- Genuinely well-built IDE (VS Code fork with Claude integrated).
- Hooks system automates the boring work — file save / PR open triggers.
- Spec-first inverts Cursor's model: spec is source of truth, code is build artifact.
- AWS-native by design — for AWS-first teams, "productivity multiplier."
- Multi-model routing between Claude Sonnet (reasoning) and Amazon Nova (throughput).
- Three-document format (requirements / design / tasks) creates auditable trail.

**What people complain about (direct quotes):**

- *"Files not appearing in context until IDE restart, trusted commands consuming credits without executing, and a rigid Requirements → Design → Task List → Coding pipeline that... 'kills momentum during iteration.'"* (Augment Code review, Mar 2026)
- *"Free tier of 50 credits burns through in a single session."* (Augment Code review)
- *"Documented reports of hallucinated or fabricated outputs."* (subreddit reports)
- AWS lock-in by design — Claude Sonnet 4.0/4.5 + Opus only, via Bedrock. Multi-cloud orgs find this a liability.
- "Vibe too hard, brought down AWS" — incident in early access where Kiro-generated code triggered a localized AWS service disruption.
- August 2025 pricing bug: *"some tasks are inaccurately consuming multiple requests... causing people to burn through their limits much faster than expected."* (AWS Director of Product Management)
- New pricing structure (vibe vs spec requests, $0.04 / $0.20 per request over limit) — *"did not go down well with developers, with several of them taking to social media and varied forums to express their disappointment."* (InfoWorld, Aug 2025)
- *"After two days of iterative changes on the OAuth2 refactor, my requirements.md described a feature that no longer matched the code."* (Augment Code review — spec drift)
- Mixing client tenants in one workspace = IP leakage risk.
- No MCP support as of April 2026 (Claude Code has it).
- Spec overhead is friction for simple tasks.
- 50 interactions/month free tier "runs out fast."

**What Rapoport Studio does differently:**

| Kiro problem | Studio answer |
|---|---|
| AWS lock-in | We're cloud-agnostic at the artifact level. Cloudflare Workers + Supabase is our deploy stack, but the OpenSpec corpus is portable to anywhere. Client owns the corpus on full payment per IP-B. |
| Model lock-in (Bedrock-only) | Muse uses Anthropic SDK directly. Easy to swap to OpenAI / Mistral / open-weights. The corpus shape doesn't change. |
| Mixing tenants = IP leakage | **Multi-tenant by structural design.** Each engagement is its own canvas + own openspec corpus, separated by RLS at the database layer. Per `mirror.md § Visibility model`, cross-slug isolation is the central rule, with row-level security + UI filtering as defense in depth. |
| Pricing surprises | €5k Discovery is fixed-fee, not metered. No "credit drained by buggy task" failure mode. |
| Rigid pipeline kills iteration | Phase-1-design / Phase-2-closing / Phase-3-implementation is *advisory*, not gate-enforced. Pavel's reply pattern (`a` / `C` / `??`) over-rides phase boundaries any time. |
| Spec drift | `forge audit` + archive convention + decisions.md with `superseded_by` keep drift visible. Drift is documented, not silent. |
| No MCP | Studio's Muse runtime is built on top of Anthropic SDK with native tool-use — equivalent capability without the Bedrock detour. |

### 2.4 BMAD-Method (~10k+ stars, MIT, full lifecycle)

**What people love.**

- Multi-agent simulation of full agile team — Analyst, PM, Architect, Dev, QA personas.
- Adversarial code review (`/bmad-bmm-code-review`) catches issues a standard review would miss.
- "Party Mode" runs multiple agent personas through design before implementation.
- Best fit for *"large, enterprise-scale projects where you need a comprehensive, highly structured, and multi-agent approach."* (nosam.com review)
- BMAD Quick mode skips straight to implementation when full process is overkill.

**What people complain about (direct quotes):**

- *"Cochran (2025) estimates that approximately two months are required to master advanced techniques."* (Anderson Santos, Jan 2026)
- *"Twelve agents, a heavy artifact set and a steep learning curve."* (ranthebuilder.cloud, Apr 2026)
- *"`/bmad-help` is available but rarely needed... It mostly exists to rescue you from BMAD's own complexity."*
- *"The method demands understanding of CLI commands, YAML configuration files, and the roles and handoffs among approximately six to seven agent personas."*
- *"Day-to-day usage can be labor-intensive... requiring developers to switch contexts frequently between web interfaces for planning and IDE environments for coding."*
- *"Excessive for small projects or isolated fixes."*
- Plan-everything-first philosophy *"can feel inflexible, particularly for exploratory or rapidly evolving projects."*
- Heavy artifact set "is hard to review."

**What Rapoport Studio does differently:**

BMAD is the closest thing to what the studio does conceptually (multi-role thinking applied to software), but at the framework level rather than the service level.

| BMAD problem | Studio answer |
|---|---|
| 2-month learning curve | Client never learns the methodology. Studio operates the methodology *for* the client; client receives the artifact. Client onboarding for IP-B engagement is 1 hour, not 2 months. |
| 12 agents to coordinate | One canvas, one Muse, one specialist (or Pavel) at a time. The 5-persona research system in Muse is *adversarial review*, not parallel agent simulation. Less ceremony, comparable adversarial value. |
| Web↔IDE context switching | Single canvas surface (`app.rapoport.studio/canvas/[id]`). Decision rows, file edits, agent runs all in one place. |
| Excessive for small projects | Discovery €5k is sized for "I have a problem and need clarity," not "I'm building an enterprise platform." Engagement scope is the variable. |
| Plan-everything-first inflexibility | Phase-1-design is exploratory, *non-gating*. Pavel keeps `[DECISION]` / `[DECISION-CANDIDATE]` / `[OPEN]` / `[PENDING]` markers in deep-dives that never need to close into a proposal. |
| Heavy artifacts hard to review | Capability files have `Status` markers, dependency graph is one query, maturity scorecard is one grep. Reviewable at a glance. |

### 2.5 Augment Code Intent (commercial, platform)

**What people love (vendor-positioned but credible).**

- Living specs that update continuously rather than on refresh.
- Multi-agent orchestration with isolated git worktrees.
- SOC 2 Type II + ISO/IEC 42001 certifications — passes enterprise procurement.
- 400,000+ files context engine — handles large codebases.

**What people complain about.**

- Public beta, macOS-only.
- Augment Code subscription required for full capabilities (paid).
- *"Some of its strongest claims still rely on vendor documentation rather than independent benchmarks."* (the vendor's own competitor analysis admits this)
- Vendor lock-in if used as platform vs framework.

**What Rapoport Studio does differently:**

Intent is a **platform** (continuous service); studio is an **engagement** (bounded work that produces an artifact). Different products. The studio's specialist-network model addresses the same "compliance + multi-agent + governance" axis but with humans + agents + service, not platform-as-a-service.

We are weaker than Intent on:
- Continuous monitoring (we don't run between engagements unless retained).
- Enterprise certifications (no SOC 2 yet).
- Platform-grade tooling support.

We are stronger than Intent on:
- Artifact ownership (client owns deliverable).
- Methodology portability (client keeps openspec corpus, can switch to any other CLI).
- Cost (one-time engagement vs subscription).
- Multi-cloud (Intent is macOS-only desktop; we work where the client deploys).

### 2.6 Cursor / Claude Code / Antigravity (no spec layer)

These are AI-IDEs without structured specifications. Users get fast iteration, no upfront discipline, and *"intent stays fragmented across task streams. There is no persistent specification layer governing what agents should do when their outputs interact."* (Augment Code, Apr 2026)

**What Rapoport Studio does differently:**

Different category. Cursor and Claude Code are *coding tools*; we are a *methodology service that produces specs to feed those tools*. Our clients can use any of these alongside the studio engagement — we ship specs that teams then implement using whatever IDE they want.

This is actually the right way to position the relationship: *Cursor and Claude Code work better against a Rapoport-Studio-produced spec than against a vibe-coded prompt.* No competition; complementary.

---

## 3. Studio-vs-studio — what about consultancies, not tools?

### 3.1 Thoughtworks — the gold standard

**What people praise (and Thoughtworks itself reports):**

- 30+ years, 10,000+ employees, 47 offices, 18 countries.
- Tech Radar — biannual industry report, ~75% read by senior engineers in target enterprises.
- ISO 27001 certified across all global offices.
- Recently launched AI/works platform.

**What the data actually says:**

From their own February 2026 report with IDC (500 senior IT decision-makers surveyed):

- *"While AI adoption is widespread among large enterprises, operational maturity lags significantly behind. While nearly nine in 10 organizations have adopted AI tools, the vast majority (approximately 90%) remain stuck in reactive modernization cycles. Only 12% have managed to break free and transition to a fully continuous, AI-driven optimization model."* (Thoughtworks/IDC, Feb 2026)

From their May 2025 report:

- *"Only 17% of organizations are true Leaders in digital and AI readiness."* — Even with Thoughtworks-level consultancy support across an industry, 83% are not leading.

The honest read: **traditional methodology consultancies report widespread structural failure of their own clients.** This is not Thoughtworks' fault — it's a category-level problem with project-based methodology consulting:

- Engagement ends; methodology leaves with the consultants.
- Client lacks the persistence layer to keep the methodology alive.
- "Reactive modernization cycles" = consulting engagement → drift → consulting engagement → drift.

**What Rapoport Studio does differently:**

The studio's IP-B model is structurally designed against the modernization-cycle problem:
- The methodology corpus stays with the client when the engagement ends.
- The corpus is plain markdown — readable by any future engineer, any future LLM, any future tool.
- "Continuous modernization" is what `current/` + `archive/` makes possible by default — the corpus is the artifact that prevents drift between engagements.

**Where Thoughtworks is genuinely better:**

- Scale (we have one engagement; they have hundreds running concurrently).
- Industry depth (vertical specialization).
- Procurement weight.
- Tech Radar reach.

We're not playing in the same league. The honest positioning: *"For founder-stage and SRL-stage engagements where Thoughtworks won't take the call, the studio offers methodology-grade work with the artifact taken away."*

### 3.2 37signals / Basecamp — closest spiritual sibling

**What people love (and what's measurable):**

- 30+ years of public manifesto continuity.
- Public handbook on GitHub (basecamp/handbook).
- Books read across industry: Getting Real, REWORK, REMOTE, Shape Up, It Doesn't Have to Be Crazy at Work.
- Built an audience pre-product (manifesto 1999 → Basecamp 2004).
- Created Ruby on Rails as side-effect.

**What people complain about:**

- They're a *product company*, not a services consultancy. The handbook is marketing for Basecamp/HEY, not a service offering you can engage.
- Their methodology (Shape Up) is opinionated and works for them — translating it to other contexts requires real adaptation.
- DHH's public positions occasionally controversial (Basecamp 2021 internal politics events).

**What Rapoport Studio does differently:**

37signals is the **closest spiritual sibling** in the studio's reference set, but at a different layer:

- 37signals built a *product company* with a public methodology as marketing.
- Rapoport Studio is building a *services studio* with a public methodology as the deliverable shape.

The studio can borrow the **publishing pattern** (public charter, public handbook, public manifesto) without competing with the product. 37signals proves the pattern works — across 25+ years of continuous publishing.

### 3.3 Israeli AI build-shops (ShowcaseIT, AI Superior, Webeet, Globaldev, Intersog)

Already covered in the Israeli-ecosystem companion file (see frontmatter). Summary: *"the build-shop AI agencies frame AI-integration as their build, not the founder's spec. The 'we'll design the AI layer, you keep the spec' pitch is still empty."*

Direct quote from category review (DesignRush, May 2026): *"What stood out most was their ability to combine technical precision with strong product empathy."* — what's missing in this praise is any mention of methodology, spec ownership, or post-engagement client autonomy. It's craftsmanship-as-a-service, not methodology-as-a-service.

### 3.4 BCG / PwC / EY / Infosys / IBM AI consulting

The Harvard Business Review September 2025 framing:

- *"AI is transforming consulting by making it possible to automate tasks traditionally handled by junior consultants such as research, modelling, and analysis. This shift is leading to a new leaner consulting model, the 'obelisk,' which features fewer layers and smaller teams."*
- The new model has three roles: AI facilitators (trained in latest AI tools and data pipelines), engagement architects (define problems, interpret AI outputs), client leaders (deep trusted relationships with senior execs).

**What this means:** the entire consulting industry is restructuring around exactly the role taxonomy Rapoport Studio is built around. The studio is *prematurely structured for the post-AI consulting model* — small team, AI-native delivery, methodology-as-deliverable.

The HBR framing validates the studio's structural choices five years ahead of the curve. What it doesn't help with: distribution. BCG and PwC have warm-intro density measured in tens of thousands; the studio has it measured in single digits.

---

## 4. The thing none of them does that we do

After all the per-tool and per-studio breakdown, three things emerge as **structural gaps in the entire competitive landscape** that the studio uniquely addresses:

### 4.1 Methodology + service + artifact, all together

| | Methodology | Service (engagement) | Artifact (client owns) |
|---|---|---|---|
| Spec Kit | ✓ | ✗ | partial (templates) |
| OpenSpec | ✓ | ✗ | partial |
| Kiro | ✓ | ✗ | ✗ (locked to AWS) |
| BMAD | ✓ | ✗ | partial |
| Intent | ✗ (platform) | partial | ✗ (platform) |
| Thoughtworks | ✓ | ✓ | ✗ (engagement ends) |
| 37signals | ✓ | ✗ | ✗ (product, not service) |
| **Rapoport Studio** | ✓ | ✓ | ✓ |

The studio is the only entry that ships all three. This is the load-bearing positioning claim.

**Tension flagged 2026-05-07:** [RAP-119](../../archive/2026-05-08-add-team-onboarding-canvas/) is the studio's first multi-author engagement and runs as **four equals co-authoring a product**, not the canonical studio↔client pair. The IP-B model assumes "client owns deliverable, studio retains methodology." When there is no single client — when artifact ownership is collective — the all-three claim needs either a "collective-client" template or a caveat. Empirical resolution lives in [`rap-119-as-research-bed.md`](./rap-119-as-research-bed.md) § 3.4. Until D9 (legal entity, deferred per RAP-119 `design.md`) resolves, public commitment to the all-three claim should carry an explicit caveat for collective-authorship engagements.

> **Update 2026-05-08 — RAP-119 cancelled.** Engagement deprioritised
> against MVP v1 critical path; collective-authorship empirical
> resolution did not arrive. Caveat for collective-authorship
> engagements still required when public commitment lands; pending an
> alternative bed (next multi-author engagement). D9 remains
> unresolved.

### 4.2 The contributor model (specialist network with named attribution)

None of the SDD tools has a built-in concept of "this paragraph was authored by this specialist." None of the consultancies open-sources their contributor model. The studio's `specialist-network.md § 6` (contribution model) + emerging `D-NW-*` rows + Moldovan inalienable-moral-rights-grounded contracts are genuinely novel.

This intersects with the philosophical work in `authorship-as-creator-identity.md`: the studio doesn't just attribute, it commits to a public charter for emerging professions (theme steward, canvas host, contribution reviewer).

### 4.3 IP/legal architecture for AI-collaborative work

None of the SDD tools addresses what happens to authorship when AI co-drafts a spec. Thaler v. Perlmutter (Mar 2025) ruled AI cannot be an author; nobody has shipped a methodology that handles this in production. The studio's `legal-landscape-authorship.md` is a 4250-word audit of how Moldovan / EU / Israeli / US law applies — and the corpus's "AI assistance: drafted with Claude" footer convention is the operational corollary.

This is a niche right now. It will be table stakes in 2027.

---

## 5. Where we are honestly weaker

Not selling. Calling these out so the studio can decide which to attack and which to live with.

| Weakness | Honest state | What we'd need |
|---|---|---|
| **Brand recognition** | GitHub Spec Kit has 75k stars; we have ~0 public artifacts | Phase 1 of `open-source-positioning.md` — public methodology repo |
| **Engagement track record** | Zero archived non-Pavel engagements *(as of 2026-05-08: [RAP-119](../../archive/2026-05-08-add-team-onboarding-canvas/) cancelled — multi-author engagement scaffolded but never ran; first non-Pavel datapoints still pending an alternative engagement)* | Andrei + Gleb + 1 other non-self engagement archived to publish case studies |
| **Procurement weight** | No SOC 2, no ISO 27001, no AWS partner status | Probably not worth chasing pre-revenue |
| **Tech Radar / category authority** | Thoughtworks' Tech Radar is read by ~75% of senior engineers in target enterprises; we have an OpenSpec corpus | Be cited *in* a Tech Radar (not produce one) |
| **Multi-engagement velocity** | One Pavel = one canvas at a time *(as of 2026-05-08: RAP-119 1:3+1 canvas cancelled before kickoff; loop-protocol scaling test pending — protocol survives at [`./loop-protocol.md`](./loop-protocol.md), validation-pending)* | Specialist network has to actually work — Q-NW-1 to Q-NW-5 still open |
| **Continuous post-engagement support** | Engagement ends, studio leaves; no retainer model yet | Possibly a Phase-2 product (subscription methodology updates) |
| **No platform** | We ship corpora, not running infrastructure | Likely structural — we're a service, not Augment Code |

---

## 6. One-paragraph positioning narrative

Suitable for `rapoport.studio/services` copy and the philosophy section of the studio charter:

> Rapoport Studio operates the methodology layer above spec-driven development tools like OpenSpec, Spec Kit, and Kiro. Where those tools answer *how* to write specs, the studio answers *who writes them, under what contract, with what governance, and what the client takes home*. Each engagement produces a portable methodology corpus the client owns, structured around a charter that names contributors, an IP model that separates deliverable from methodology, and a contribution model that treats authorship as the legal floor and the philosophical foundation. We are not another CLI. We are not a platform. We are a small studio that publishes its operating system and runs engagements against it.

Three honest disclaimers attached:

- We are early. First non-self engagement archives 2026.
- We use Fission-AI/OpenSpec under the hood and are open about it.
- We do not compete with Thoughtworks, GitHub, or AWS at scale. We compete for the engagements they decline.

---

## 7. Open questions

| ID | Priority | Question |
|---|---|---|
| `Q-CP-1` | P0 | Do we publicly position against Spec Kit / OpenSpec / Kiro / BMAD by name, or implicitly? Recommendation: **implicitly**. Naming competitors small-up is rarely worth the goodwill cost; the comparison reaches the right reader through "we use OpenSpec, we add the methodology layer above it" framing. |
| `Q-CP-2` | P0 | Does the studio commit to the "all three: methodology + service + artifact" claim publicly? It's defensible only if it stays true — i.e. studio never pivots to platform-as-a-service. |
| `Q-CP-3` | P1 | Is the obelisk-model positioning (HBR Sep 2025) something the studio cites in public copy? Argument for: validates structural choices. Argument against: ties studio to a frame that may age poorly. |
| `Q-CP-4` | P1 | Where does Intent (Augment Code) sit in the studio's mental map? Adjacent platform, not direct competitor — but might become one if Augment Code adds services. |
| `Q-CP-5` | P2 | Should the studio publish a comparison table like §2 publicly, or keep it internal? Internal-only avoids vendor-reaction risk; public adds clarity to procurement-stage prospects. Recommendation: internal until first 3 engagements archived. |
| `Q-CP-6` | P2 | What's the studio's response when a prospect asks "why not just use Spec Kit/OpenSpec ourselves?" Recommendation: "you can, and you might. We add the contributor model, the contract layer, and the artifact discipline. If you don't need those, you don't need us." |

---

## 8. How this thread closes

When the studio ships Phase 1 of [`open-source-positioning.md`](./open-source-positioning.md) (public methodology repo) **and** archives at least 2 non-Pavel engagements that can be cited as case studies. At that point:

- The positioning narrative has public artifacts to anchor against.
- The "we are not another CLI" claim is verifiable (people can read the corpus).
- The "client owns the deliverable" claim is provable (case studies show the artifact in the client's repo).

Until then this file is research, not positioning. Used internally to inform copy; not used externally to make claims the studio can't yet verify.

---

## 9. Adjacent prior thinking

### 9.1 Existing in repo (links resolve today)

| Source | Relationship |
|---|---|
| [`open-source-positioning.md`](./open-source-positioning.md) | What to publish (methodology repo, license choices). This file informs *what to say* once published. |
| `studio-charter.md § 1` (AI as orchestration substrate) | The thesis being differentiated against the SDD-tool category. |
| `specialist-network.md § 6` | Contributor model — the unique-to-studio structural advantage in §4.2. |
| `mirror.md § Visibility model` | Multi-tenant isolation — the structural answer to Kiro's "mixing client tenants in one workspace" gap (§2.3). |
| [`rap-119-as-research-bed.md`](./rap-119-as-research-bed.md) | Observation protocol that wires § 5 weakness rows ("engagement track record", "multi-engagement velocity") and § 4.1 collective-client tension to RAP-119 surfaces. First empirical test of multiple positioning claims. |
| [`openspec/archive/2026-05-08-add-founder-track/research/israeli-startup-ecosystem.md`](../../archive/2026-05-08-add-founder-track/research/israeli-startup-ecosystem.md) | Israeli founder ecosystem — different scope. Listed in frontmatter as companion; promotion to `_methodology/research/` is a separate decision. |

### 9.2 Forward references — files cited but not yet drafted

This thread references two sister files that do not yet exist in the repo. References are kept intact so cross-links resolve when those files are integrated.

| Cited file | Where referenced | Current status |
|---|---|---|
| `legal-landscape-authorship.md` | § 2.2 (`Q-LL-1` gating for contract template); § 4.3 (entire claim about IP/legal architecture rests on this 4250-word audit) | Not yet drafted. The §4.3 "structural gap nobody else fills" claim is **load-bearing on a phantom file**. The strength of the positioning argument cannot be independently verified until this file exists. |
| `authorship-as-creator-identity.md` | § 4.2 (philosophy frame for contributor model — "naming/propagation/legitimacy"); §4.2 references "theme steward, canvas host, contribution reviewer" as named professions | Drafted in chat 2026-05-07 with five integration questions outstanding; not yet placed in repo. |

### 9.3 Open loops — claims that depend on unverified data

| Loop | Resolution path |
|---|---|
| GitHub Spec Kit Discussions (#1482, #152, #1686) direct quotes | Quotes are presented as verbatim. Not independently verified by the integration pass. **Verify before any public Phase 1 publication.** |
| Augment Code review claims (Spec Kit overhead, Kiro spec drift) | Vendor's competitor analysis. Useful but vendor-positioned. Treat as one-source-with-bias. |
| Thoughtworks/IDC Feb 2026 statistics ("90% stuck in reactive modernization cycles", "Only 12%...") | Cited from press release. Verify direct figures against the underlying IDC report before public citation. |
| HBR Sep 2025 "obelisk model" | Real article. Verify exact framing before quoting in public copy — paraphrase is faithful but the "obelisk" term should be quoted only if HBR uses it verbatim. |
| Sources in § 10 generally | Several reviews dated Mar–Apr 2026 (Augment Code, ranthebuilder.cloud, Big Hat Group, Anderson Santos) — within current-date window but not independently spot-checked here. Verify before any external citation. |

---

## 10. Sources cited

| Source | Reference |
|---|---|
| GitHub Spec Kit Discussion #1482 (maintenance + memory concerns) | github.com/github/spec-kit/discussions/1482 (Jan 2026) |
| GitHub Spec Kit Discussion #152 (specs as session deltas) | github.com/github/spec-kit/discussions/152 (Oct 2025–Mar 2026) |
| GitHub Spec Kit Discussion #1686 (High Level Design Concerns) | github.com/github/spec-kit/discussions/1686 (Feb 2026) |
| Spec Kit Review (Vibe Coding App) | vibecoding.app/blog/spec-kit-review (Jan 2026) |
| Augment Code Spec Kit vs Intent comparison | augmentcode.com/tools/intent-vs-github |
| Augment Code 6 Best Kiro Alternatives | augmentcode.com/tools/best-kiro-alternatives (Mar 2026) |
| Augment Code Kiro vs Intent | augmentcode.com/tools/intent-vs-kiro |
| Augment Code Kiro vs Antigravity | augmentcode.com/tools/kiro-vs-antigravity |
| Kiro pricing bug coverage | infoworld.com (Aug 2025) |
| Kiro vibe-coding incident summary | openaitoolshub.org/en/blog/kiro-review-amazon-ide |
| Claude Code vs Kiro comparison | lowcode.agency/blog/claude-code-vs-kiro |
| AWS Kiro testing review | thenewstack.io (Mar 2026) |
| Anderson Santos, "You should BMAD — part 2" (critique) | adsantos.medium.com (Jan 2026) |
| Ran the Builder, "Three Spec-Driven AI Tools tested" | ranthebuilder.cloud/blog (Apr 2026) |
| Nosam, "OpenSpec vs Spec-Kit vs BMAD" | nosam.com (Nov 2025) |
| Hightower, "GSD vs Spec Kit vs OpenSpec vs Taskmaster AI" | medium.com/@richardhightower (Feb 2026) |
| Big Hat Group, "OpenSpec vs Spec Kit" | bighatgroup.com (Mar 2026) |
| Thoughtworks/IDC report on AI modernization | prnewswire.com / morningstar.com (Feb 2026) |
| Thoughtworks State of Digital and AI Readiness | businesswire.com (May 2025) |
| Thoughtworks Tech Radar Vol 33 | prnewswire.com (Nov 2025) |
| HBR, "AI Is Changing the Structure of Consulting Firms" | hbr.org/2025/09/ai-is-changing-the-structure-of-consulting-firms |
| 37signals manifesto (1999) | 1999.37signals.com/manifesto |
| 37signals current site (81 signals) | 37signals.com |
| Basecamp Handbook | basecamp.com/handbook + github.com/basecamp/handbook |
| Fission-AI OpenSpec | github.com/Fission-AI/OpenSpec |
| GitHub Spec Kit | github.com/github/spec-kit |
| AWS Kiro | kiro.dev/docs/specs |
| Cursor | cursor.com |
| Claude Code | claude.ai/code |
| Augment Code Intent | augmentcode.com |

---

## 11. Change log

| Date | Author | Change |
|---|---|---|
| 2026-05-07 | Pavel (founder) | Opened thread. Seven-source-cluster pass: Spec Kit complaints (incl. maintenance + memory issues), OpenSpec gaps (governance + brand), Kiro problems (AWS lock-in + pricing + drift), BMAD critique (learning curve + complexity), Augment Intent platform analysis, Thoughtworks/IDC enterprise data, HBR obelisk-model framing. Six open questions Q-CP-1 to Q-CP-6, P0 on naming-vs-implicit positioning and the methodology+service+artifact public claim. AI assistance: drafted with Claude (Anthropic). |
| 2026-05-07 | Integration pass | Placed at `_methodology/research/competitive-positioning-sdd-frameworks.md`. Companion-file reference adjusted: cited file lives at `openspec/archive/2026-05-08-add-founder-track/research/israeli-startup-ecosystem.md`, not under `_methodology/research/`. Forward-references to two sister files (`legal-landscape-authorship.md`, `authorship-as-creator-identity.md`) retained intact and surfaced in § 9.2; § 4.3 claim flagged as load-bearing on a phantom file. Source-verification flags surfaced in § 9.3. |
