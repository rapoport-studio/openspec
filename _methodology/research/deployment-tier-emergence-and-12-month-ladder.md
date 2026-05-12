# Deployment-tier emergence (May 2026) and Rapoport Studio's 12-month ladder

> **Status:** open research, event-triggered strategic synthesis. Captures the structural shift announced on 2026-05-11 (OpenAI Deployment Company + Anthropic / Goldman Sachs / Blackstone $1.5B fund + Tomoro acquisition) and translates it into a 12-month evolution ladder for Rapoport Studio.
> **Owner:** Pavel (founder).
> **Authors:** Pavel Rapoport <pavel@rapoport.studio> (founder, primary); Claude Opus 4.7 (Anthropic) — event sourcing from public announcements, peer-studio survey via web search (LumentAI, Synthwave Studio, DeepKlarity cluster), structural framing of the 7-layer deployment stack, drafting.
> **Method:** Single-pass, event-triggered. Public announcement sourcing (OpenAI / Anthropic press, secondary trade-press synthesis) on the May 11 events. One web-search pass to verify Tessl product status (Sept 2025 launch, Spec Registry open beta) and to surface peer-shape AI engineering studios (Montreal, Zurich clusters). No first-party interviews. No quantitative benchmark. Structural framing extends — does not replace — the regional landscape work in [`competitor-landscape-synthesis.md`](./competitor-landscape-synthesis.md) and the SDD-tool comparison in [`competitive-positioning-sdd-frameworks.md`](./competitive-positioning-sdd-frameworks.md).
> **Reusable for:** internal positioning copy, planning conversations, future home-page revision triggered by a Stage-5 decision, optional Lab MDX article on "the deployment-tier shift". Authorship: open.
> **Opened:** 2026-05-12.
> **Companion to:** [`competitive-positioning-sdd-frameworks.md`](./competitive-positioning-sdd-frameworks.md) (tool-layer SDD comparison), [`competitor-landscape-synthesis.md`](./competitor-landscape-synthesis.md) (regional landscape MD/RO/IL/EU), [`open-source-positioning.md`](./open-source-positioning.md).
> **Feeds into:** [`openspec/changes/lock-positioning-trajectory/`](../../changes/lock-positioning-trajectory/) (the change that codifies the framework derived from this research); quarterly strategic review; Stage-5 fork decision (~Q2 2027); `studio-charter.md` if Studio shape changes.

---

## 1. Why this file exists

Two existing research files cover adjacent terrain:

- [`competitive-positioning-sdd-frameworks.md`](./competitive-positioning-sdd-frameworks.md) — tool-layer comparison (Spec-Kit, OpenSpec, Tessl, BMAD, Kiro, Augment) and methodology-consultancy gap.
- [`competitor-landscape-synthesis.md`](./competitor-landscape-synthesis.md) — regional competitive landscape across Moldova, Romania, Israel, Europe.

Neither captures the **deployment-tier emergence** triggered on 2026-05-11, nor the **forward-looking ladder** for Studio's own evolution over the next 12 months. This file fills both.

It is:

- A **structural read** of where capital and FDE-class talent are flowing in the AI-services market post-May-2026.
- A **forward operating plan** with six concrete stages and a Stage-5 development fork.

It is not:

- A regional competitor refresh — that lives in the four regional files.
- A new ICP decision — Muse-first founder-track is already locked per `competitor-landscape-synthesis.md` § 3.C.
- An OpenSpec change proposal — this is research, the proposal that consumes it is [`openspec/changes/lock-positioning-trajectory/`](../../changes/lock-positioning-trajectory/).

---

## 2. The 2026-05-11 signal

Three announcements on the same day. Treat them as one event.

### 2.1 OpenAI Deployment Company (DeployCo)

- **Capital:** $4B raised at $10B pre-money, OpenAI retains majority. Investors get 17.5% minimum return floor; profits capped.
- **19 founding partners** including Bain & Company, Capgemini, McKinsey & Company, plus capital partners TPG, Advent, Brookfield.
- **Operating model:** Forward Deployed Engineers (FDEs) — a sales-and-delivery shape pioneered by Palantir — physically embedded inside Fortune 500 client teams. Per CRO Denise Dresser: *"engineers literally sit next to the team, learn the workflows, and help wire AI capabilities into internal applications."*
- **First acquisition:** Tomoro — UK-based applied-AI consultancy, ~150 engineers, prior clients include Virgin Atlantic (AI concierge) and Tesco. Tomoro is being absorbed into the FDE bench.

### 2.2 Anthropic + Goldman Sachs + Blackstone $1.5B fund

- Announced one week prior to DeployCo.
- Fund vehicle, not an FDE army — designed to **finance** AI adoption inside hundreds of mid-market and enterprise companies that already have or are evaluating Claude.
- **Goldman Sachs is the only investor in both DeployCo and the Anthropic fund.** Read this as the financial market hedging across both deployment paths.

### 2.3 Brad Lightcap (COO OpenAI) framing

> *"Customers tell us they need help going from pilot to production. Deployment Company will put our engineers inside their teams with the resources to actually ship."*

This is the clearest public statement that the bottleneck has moved. **2024–2025 was about model quality. 2026 is about integration, change management, security, and process redesign.** Both OpenAI and Anthropic are descending one layer in the stack — from "we sell you the model" to "we sell you the deployment".

---

## 3. The deployment stack — seven layers

The market for AI services in 2026 is not a flat field. It stratifies into seven layers, with sharp differences in capital structure, ticket size, buyer profile, and vendor lock.

| Layer | Occupants (2026) | Ticket | Buyer | Vendor lock |
|---|---|---|---|---|
| **L1** Foundation models | OpenAI · Anthropic · Google DeepMind · Meta · xAI | n/a (sold below) | ML platform teams | very high |
| **L2** Cloud + model API | Azure OpenAI · AWS Bedrock · Google Vertex · Anthropic API direct | per-token | platform engineers | high |
| **L3** **Deployment infra (NEW 2026)** | **OpenAI Deployment Company** · **Anthropic + GS / Blackstone fund** | $500k–$10M+ /year | C-suite, CIO | very high (model + infra + people + methodology) |
| **L4** Big consulting / global SI | McKinsey QuantumBlack · Bain (now also DeployCo partner) · BCG X · Accenture · Capgemini (now also DeployCo partner) · Deloitte · IBM Consulting | $2M–$100M+ /engagement | board / C-suite | high (methodology lock + multi-year transformation) |
| **L5** Applied-AI boutiques (50–200 people) | Tomoro **[acquired by OpenAI 2026-05-11]** · Sona · Aigency · regional equivalents | $200k–$2M /project | mid-market VP | medium |
| **L6** **Curated-team studios (← Rapoport Studio)** | Independent boutique studios with proprietary operating model + AI substrate | €5k Discovery → €50k–€150k engagement | founders, SMB, regional mid-market | minimum (artifact stays in client repo) |
| **L7** Local dev shops + freelancers | Toptal · Upwork · Moldovan / Romanian / Israeli regional agencies | €2k–€30k /project | owner, operations lead | medium (people lock — talent leaves, knowledge leaves) |

**The new attack zone is L3–L5.** That's where the May 11 capital is flowing. L4 (big consulting) is being squeezed from above (DeployCo's FDEs) while continuing to fight L5 from below. L5 is being bought (Tomoro is the first; expect more in 12–18 months).

**Rapoport Studio sits at L6.** That position is defensible in a 12–24 month horizon precisely because the L3–L5 capital flow does not yet reach the geographies (MD/RO/IL) and ticket sizes (€5k–€150k) where Studio operates. Beyond 24 months, L3 distribution channels will reach those geographies via local SI partners — see § 9 (monitoring).

---

## 4. Three competitor types for Rapoport Studio

The single most important re-framing this file introduces: **Studio's competitors split into three structurally distinct categories**, each requiring a different response. Conflating them produces wrong moves.

### 4.1 Peer-shape: studios at our scale and shape

These are the players we lose to (or beat) on a specific contract. Small (1–10 people), AI-engineering-focused, sometimes spec-driven, often touting "senior practitioners only."

| Player | Origin | Shape | Threat to Studio |
|---|---|---|---|
| **Tessl** | UK · Guy Podjarny (ex-Snyk) | Product, not service. Spec-first AI dev framework. Launched Sept 2025; Spec Registry open beta with 10,000+ pre-built specs. | **Methodology rival.** Same thesis, different business model. See § 5.4. |
| **LumentAI** | Montreal | 2-person senior team; autonomous agents; 30+ delivered projects. | Direct global English-speaking peer. |
| **Synthwave Studio** | Zurich | 5–8 people; chatbots + automation; "<2-week MVP" claim, 50+ shipped, 98% retention. | Direct EU peer. Likely higher hourly rate than Studio. |
| **DeepKlarity / Parallel Studio / Brobot** | NA + EU cluster | Each 1–5 people; "end-to-end AI capabilities, no juniors" pitch. | Cluster — the "AI engineering studio" category is filling rapidly. |
| **Israeli AI dev shops (cluster)** | Tel Aviv | 3–15 people; ex-8200 / ex-Mobileye talent; warm-intro distribution; less spec-driven, more agile. | **Most dangerous on Founder Track.** Their geo-advantage in IL exceeds ours. |
| **In-house technical co-founder (default)** | Anywhere | Founder writes everything themselves on Cursor + Claude + Lovable. | The default not-decision for half of bootstrapped IL founders. |

### 4.2 Substrate-eaters: AI builders that compress the bottom of our funnel

These do not compete with Studio for any specific contract. They change client behavior — by making the first phase of work cheaper-than-free, they erode the funnel into Discovery.

| Player | What it eats | Threat |
|---|---|---|
| **Lovable** (Sweden, YC) | Non-technical founder MVP phase | Removes the first €5k of our Founder Track funnel. PMF check ≠ needs Studio. |
| **Bolt.new** (StackBlitz) | Technical founder prototype phase | Same dynamic; better-quality output for code-comfortable founders. A good Bolt prototype could become Studio input, not a substitute. |
| **v0 (Vercel) · Replit Agent · Devin (Cognition)** | Default starter tooling for any new project | Changes the *default way to begin a project*. Within 12 months, Discovery must do something these structurally don't. |

**Key point:** competing with these tools on speed-of-MVP is a losing economic game for any service business. Studio's response is asymmetric — sell what they can't sell (artifact ownership, change-aware spec, network curation), not faster MVPs.

### 4.3 Methodology rivals: who claims the "spec-driven dev" standard

Already covered in depth in [`competitive-positioning-sdd-frameworks.md`](./competitive-positioning-sdd-frameworks.md). Net update from the May 11 events:

- **Tessl** continues building its Spec Registry. If it becomes a de facto standard, OpenSpec in a Studio client's repo starts to look exotic (cost, not value).
- **GitHub Spec-Kit** has stalled (per the Discussion #1482 quote in the SDD-frameworks file: original PM left for Anthropic; 22 PRs open, 0 commits Jan 2026). One year from now this could be revived under Microsoft enterprise muscle, or it could be effectively abandoned.
- **Anthropic Skills** is complementary, not competitive — Skills is the unit of capability, OpenSpec is the unit of engagement contract. The two compose.

This file does not duplicate the SDD comparison; refer to the companion file for tool-by-tool teardown.

---

## 5. What each competitor emphasizes (tech / marketing / user)

Three-axis read of the categories above. This is the field map for "what *not* to copy" as much as "what to learn from."

### 5.1 Technology emphasis

- **Tessl:** spec → automated implementation as a product. SDD workflow with approval gates.
- **Peer studios (LumentAI, Synthwave, DeepKlarity cluster):** "end-to-end AI capabilities" — vague, held together by founder seniority claims.
- **Substrate-eaters:** prompt → working app, fastest possible.
- **DeployCo:** FDE labour + OpenAI proprietary infra + 150-engineer Tomoro bench.
- **Studio (us):** OpenSpec + Forge + Muse — a pipeline whose deliverable is an *artifact*, not just code. Spec lives in client repo. More specific than "AI capabilities", contrastive with "prompt → app".

### 5.2 Marketing emphasis

- **Tessl:** thought leadership through Podjarny brand (ex-Snyk credibility), podcast, blog, content marketing.
- **Peer studios:** "20+ years experience" claims, retention numbers, speed claims, minimal content output, founder personal brand.
- **Substrate-eaters:** viral demos, Twitter / TikTok, freemium conversion funnel.
- **DeployCo / consultancies:** enterprise sales motion, partner channel, board-level relationships — invisible to web traffic.
- **Studio:** public `openspec/` directory as trust signal, curated network as brand asset (not "Pavel + freelancers"), `/lab` long-form. Explicitly *not* speed-claim — that's a different axis we cannot win.

### 5.3 User / buyer emphasis

- **Tessl:** senior engineers, CTOs, AI engineering teams in product companies. Self-serve.
- **Peer studios:** mid-market mid-tech companies in NA and EU.
- **Substrate-eaters:** non-technical founders + indie makers (Lovable end); technical founders (Bolt end).
- **DeployCo / consultancies:** Fortune 500 / Fortune 1000, government, board level.
- **Studio:** founders + SMB owners in MD / RO / IL. Not the sweet spot of any of the above. **This is both our moat and our ceiling.**

### 5.4 Special note: Tessl is the most important reference

Tessl bets on the same thesis Studio bets on (spec-driven AI development) but as a **product**, not a **service**. Their 10,000-spec Registry is exactly the asset Studio cannot match by working harder — it is structurally a different business model.

The defensive frame: if Tessl becomes the kubernetes of SDD by Q1 2027, Studio's OpenSpec usage benefits from the standard maturing (more tooling, more shared vocabulary). If Tessl tries to vertically integrate into services (own FDE bench), they would be a direct L5/L6 competitor. Both scenarios are plausible. **Monitor quarterly.**

---

## 6. Where Studio actually competes — by ICP

DeployCo / McKinsey / Anthropic+GS are not on our table. They play Fortune 500 at $1M+ tickets. Real competition for every Studio decision looks like this:

### 6.1 Client Track — Moldova + Romania (€30k–€150k engagement)

Real client alternatives:

- Local digital agency (low AI-orchestration ceiling)
- Toptal freelancer (no process, no methodology)
- In-house: ChatGPT Enterprise + one developer
- Do nothing

**Studio wins via:** curation + artifact ownership (spec stays in their repository). See `_methodology/rapoport-studio-engagement.md`.

### 6.2 Founder Track — Israel (€5k Discovery → €50k+ build)

Real founder alternatives:

- Solo on Cursor + Claude Code (substrate-eater path)
- Fractional CTO (€8–15k/month)
- Israeli dev shop (€80–200k MVP)
- YC-style accelerator

**Studio wins via:** the spec works without us. Founder hits PMF and leaves with working code + portable methodology. The dangerous competitor here is local Israeli AI dev shops, not DeployCo. See `archive/2026-05-08-add-founder-track/` for full Founder Track design.

### 6.3 Specialist Network — global

Real specialist alternatives:

- Toptal / Upwork (commission)
- Direct clients (no funnel)
- Hiring at DeployCo / Tomoro-style company (salary)

**Studio wins via:** curated brand — credit on `/work`, no commission, real case studies. See `_methodology/specialist-network.md`.

---

## 7. Twelve-month ladder — Stages 0 through 5

Six stages. Each has explicit state, a single-sentence proof signal, and the changes from the previous stage. Linear path through Stage 4; Stage 5 is a fork (see § 8).

### Stage 0 — Foundation ready, no clients (now, May 2026)

**State:**
- Platform: web + studio apps deployed on Cloudflare
- Forge: Architect mode active, Builder partial (RAP-48 Waves 1–3 shipped)
- Muse: substrate on Claude API
- OpenSpec: 50+ archived changes, public
- Network: model defined; no real specialists onboarded
- Revenue: 0

**Proof signal:** Self-hosting works — Studio uses its own stack daily (`migrate-studio-into-platform` shipped 2026-05-09).

**This is baseline.**

### Stage 1 — First paying engagements (~Q3 2026, +3 months)

**State:**
- 2–3 paid Discovery engagements (€5k each) — IL and MD
- 1 active full engagement (€30–50k)
- 5–8 specialists in network with real credits
- Forge Builder mode stabilized (eight known gaps from `forge.md § Known gaps` reduced to ≤2)
- `/work` carries first case studies
- Revenue: €15–60k cumulative

**Proof signal:** First Founder Track client exits Discovery into build phase. Linear shows real client tickets, not only studio-internal RAP-* issues.

**Changes from S0:** From self-hosting to paid client work. Network shifts from spec to actual people. `/work` ceases to be placeholder.

### Stage 2 — Discovery as productized SKU (~Q4 2026, +6 months)

**State:**
- Discovery (€5k, 3–4 weeks) shipping every 2 weeks
- 5–10 active engagements
- 15–25 specialists in network
- Telegram canvas surface working with real clients (per `add-tg-canvas-surface`)
- Pavel ceases to be the single bottleneck on every Discovery
- Revenue: €50–80k MRR-equivalent

**Proof signal:** Discovery is templatized — a new Discovery starts without "Pavel does everything". Pavel has a playbook plus 1–2 specialists who can lead it.

**Changes from S1:** Discovery shifts from bespoke to repeatable. Network carries the work, not new people each time. Telegram is the real operational hub.

### Stage 3 — Niche recognition in MD/IL (~Q1 2027, +9 months)

**State:**
- 15–25 completed engagements
- 30–50 active specialists
- Recognition among IL founders via warm intros, not cold pitch
- OpenSpec methodology referenced as example in external blogs
- Pavel speaking at conferences in TLV / Chișinău
- Revenue: €80–150k MRR-equivalent

**Proof signal:** Leads come without active outreach. `/contact` fills with warm intros, not ad-traffic.

**Changes from S2:** Active pitch → inbound. OpenSpec usage becomes a separate trust signal (consistent with the wedge identified in `competitor-landscape-synthesis.md` § 4 insight #8). Pavel is a visible figure in IL/MD AI ecosystem, not only Studio's founder.

### Stage 4 — Methodology beyond Studio (~Q2 2027, +12 months)

**State:**
- `openspec.dev` (or equivalent public surface) for the methodology
- Workshops / training for specialists from other studios
- Specialist certification programme operational
- 50+ active specialists, 5–10 on full-time basis
- Possibly 2–4 employees (operations, BD — not engineering)
- Revenue: €150–250k MRR-equivalent

**Proof signal:** Someone uses OpenSpec methodology (or our version of it) without Pavel's involvement. First "not-our-engagement" case study lands.

**Changes from S3:** OpenSpec moves from internal practice to public artifact. Network becomes the first contour of the business, not an add-on. Pavel begins to delegate operational layer.

### Stage 5 — Fork (12+ months, decided ~Q2 2027)

**State:**
- Decision between three paths (see § 8)
- Stage 4 produces enough data for the choice
- Any of the three is a valid exit from "founder bottleneck"
- **Do not pre-commit.** Premature choice destroys foundation that hasn't been validated by engagements.

**Proof signal:** Stage 4 has shown which path produces the most revenue, influence, or alignment with how Pavel wants to spend hours.

---

## 8. Stage 5 fork — three possible Studio shapes 12+ months out

These are mutually inhibitory choices. Pavel cannot pursue more than one as the dominant shape; secondary paths can persist as side-bets.

### Path A — Boutique destination

5–10 person studio. Premium engagements (€80k–€300k). Curated team is the product. **No** productization of methodology — it stays internal asset.

| Pros | Cons |
|---|---|
| Simple business model | Revenue ceiling ~€500k–€1M ARR |
| High margin per engagement | Heavy dependence on Pavel |
| Pavel keeps hands-on role | Vulnerable to DeployCo wave reaching MD/RO in 24 months |
| Minimum new risk | No moat beyond people |

**Fit with current trajectory: high.** Closest extension of Stage 4.

### Path B — Productized methodology

OpenSpec workflow as publicly licensed framework. Training, certification, consulting for other agencies and in-house teams. Studio engagements continue, but methodology becomes a separate revenue line.

| Pros | Cons |
|---|---|
| Real moat — own the methodology | Marketing + community-building is a separate full-time job |
| Revenue not capped by Pavel's hours | Tessl is fighting for the same standardization slot |
| Aligns with public-openspec posture | Long path to profitability |
| Snyk → DevSecOps standardization analogy | Can kill the engagement business as a side-effect |

**Fit with current trajectory: medium.** Requires Pavel to pivot into evangelist mode.

### Path C — Platform / SaaS

Internal Rapoport Studio platform (Forge + Muse + Telegram canvas surface) becomes a SaaS product for other boutique studios. "Notion + Linear + Cursor for AI-engineering teams."

| Pros | Cons |
|---|---|
| Highest ceiling if it works | Full business-model change (service → product) |
| Software margins | Requires capital or extraordinary LTV/CAC |
| Buyer is our peer (we know the pain) | At least five competitors are already building similar |
| Close to existing codebase | Service business dies faster than product grows |

**Fit with current trajectory: low.** Too far from current shape. Possible as a second move at +24 months.

---

## 9. Recommendations and non-decisions

### 9.1 Do — for the next 12 months

- Walk Stages 1–4 as written. Resist Stage 5 commitment until Q2 2027.
- Lean on three marketing axes that are structurally defensible:
  1. **Public `openspec/`** as trust signal — most peer studios in the L6 cluster don't show methodology. This is our only structural marketing differentiator.
  2. **Curated network**, not "Pavel + freelancers" — distinguishes us from "2-person studio" peers (Parallel Studio, LumentAI) the client cannot otherwise differentiate from us.
  3. **Spec ownership** — the only sentence Lovable / Bolt / DeployCo cannot copy without changing their business model.

### 9.2 Don't — for the next 12 months

- Don't rewrite the site for NA / UK enterprise. Not our ICP; we lose to Synthwave / DeepKlarity-class players there.
- Don't compete with Lovable / Bolt on speed-of-MVP. Losing economics for a service business.
- Don't announce "OpenSpec methodology" as a product before Stage 4. No proof points; reads as hype.
- Don't hire engineers as employees. Network model is stronger at our scale.
- Don't grow faster than the network can absorb. Stage 2 → 3 is the critical moment where quality breaks.

### 9.3 Monitor — single quarterly observable

**Tessl.** If by Q1 2027 they have made themselves the de facto standard ("kubernetes of SDD" — Spec Registry adopted broadly outside their own customer base), this forces the Stage 5 fork toward Path A or Path B, **not** Path C. Quarterly check is enough.

Secondary observables (annual, not quarterly):

- DeployCo expansion into EU mid-market. If their FDE bench reaches Israel via local SI partners, Founder Track positioning needs revisiting.
- Anthropic Solutions Partner programme (if launched). Studio is the right size (small, spec-driven) for downstream-implementer role in EU + Israel where Anthropic has less direct presence.
- Tomoro-style acquisitions of L5 boutiques. If two more announced in 12 months, Path A's ceiling assumption (~€1M ARR) needs revision upward — boutique consolidation increases price ceilings in adjacent markets.

---

## 10. What this research does NOT do

In the spirit of [`competitor-landscape-synthesis.md`](./competitor-landscape-synthesis.md) § 5, an honesty section:

- It does not generate leads. The Stage 1 → 2 transition still requires Pavel to write to actual founders.
- It does not write Stage 1 or Stage 2 copy. Direction is set; words are downstream.
- It does not replace the Israeli inside-partner question (open per `competitor-landscape-synthesis.md` § 7.4).
- It has a **6-month shelf life**. Re-read in November 2026 and replace if the May 11 trajectory has clearly diverged from this read. The DeployCo / Anthropic-fund situation is fast-moving.
- It does not settle Path A vs B vs C. That is a Stage 4 decision, not a Stage 0 one.

---

## 11. Cross-references

**Adjacent research in this folder:**
- [`competitive-positioning-sdd-frameworks.md`](./competitive-positioning-sdd-frameworks.md) — tool-layer SDD comparison (Tessl, Spec-Kit, BMAD, Kiro, Augment) — companion file
- [`competitor-landscape-synthesis.md`](./competitor-landscape-synthesis.md) — regional landscape synthesis (MD/RO/IL/EU) — companion file
- [`competitor-landscape-israel.md`](./competitor-landscape-israel.md), [`competitor-landscape-romania.md`](./competitor-landscape-romania.md), [`competitor-landscape-moldova.md`](./competitor-landscape-moldova.md), [`competitor-landscape-europe.md`](./competitor-landscape-europe.md) — per-region detail
- [`open-source-positioning.md`](./open-source-positioning.md) — what to publish from internal practice
- [`openspec-vs-speckit-public.md`](./openspec-vs-speckit-public.md) — public-facing OpenSpec vs Spec-Kit framing

**Methodology this feeds into:**
- [`../studio-charter.md`](../studio-charter.md) — Studio's founding charter
- [`../rapoport-studio-engagement.md`](../rapoport-studio-engagement.md) — engagement methodology
- [`../specialist-network.md`](../specialist-network.md) — network model

**Capability specs touched (read-only references, no edits triggered by this file):**
- [`../../current/platform.md`](../../current/platform.md)
- [`../../current/forge.md`](../../current/forge.md)
- [`../../current/muse.md`](../../current/muse.md)

**OpenSpec change that consumes this research:**
- [`openspec/changes/lock-positioning-trajectory/`](../../changes/lock-positioning-trajectory/) — codifies the framework derived from this file (Stages 0–5 as observable signals, Stage 5 fork criteria for Path A/B/C, drift triggers)

**Source artifacts (Cursor canvases, not in repo):**
- `~/.cursor/projects/Users-pavelrapoport-Documents-GitHub-rapoport-studio/canvases/deployco-positioning.canvas.tsx` — interactive 7-layer stack and player cards
- `~/.cursor/projects/Users-pavelrapoport-Documents-GitHub-rapoport-studio/canvases/studio-competitors-and-ladder.canvas.tsx` — interactive ladder + Stage 5 fork
- `~/.cursor/projects/Users-pavelrapoport-Documents-GitHub-rapoport-studio/canvases/positioning-proposal-preflight.canvas.tsx` — pre-flight decisions matrix that became Q-POS-1..10 in the openspec change
