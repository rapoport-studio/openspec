# Social-positioning analogs — trust × wealth-distribution evidence base

> **Status:** open research, no commitments attached.
> **Owner:** Pavel (founder).
> **Authors:** Pavel Rapoport <pavel@rapoport.studio> (founder, primary); Claude Opus 4.7 (Anthropic) — two parallel web-research subagents (~50 public-source queries combined), cross-reference against the studio's existing methodology stack, drafting and synthesis into three operational tracks.
> **Method:** Multi-pass — (a) 21 social-impact / civic-AI / public-interest / human-rights / cooperative / AI-for-good organisations surveyed from public sources (websites, ProPublica 990, GuideStar, CauseIQ, NGI, TechCrunch, organisation handbooks); (b) parallel pass on trust frameworks (Edelman Trust Barometer 2025, Mozilla Trustworthy AI, Partnership on AI ABOUT ML, Center for Humane Technology, Trust Equation) and wealth-distribution frameworks (Platform Cooperativism Consortium, ICA 7 Principles, worker-coop case data for Stocksy / Resonate / Outlandish / Hypha / Loomio / Up & Go / Drivers Cooperative, Benkler / Ostrom / Mazzucato); (c) cross-reference against `specialist-network.md`, `value-attribution.md`, `trust-gradient.md`, `positioning.md`, `lock-positioning-trajectory/`; (d) synthesis into seven repeating patterns + six rare differentiators + three operational tracks (Surface / Restructure / Bifurcate); (e) gap audit of unverified claims.
> **Reusable for:** input to `lock-positioning-trajectory` Stage 5 fork indicator sets (§ 3.1–3.3 of that change's `design.md`); input to `value-attribution.md` Q3 (value-concentration-prevention mechanism candidates); input to a future "engagement acceptance policy" doc (Data & Society template); positioning copy on `/services` and `/network`; Lab article on "what social-orientation means without NGO-vocabulary". Authorship: open.
> **Opened:** 2026-05-12.
> **Companion to:** [`./value-attribution.md`](./value-attribution.md), [`./open-source-positioning.md`](./open-source-positioning.md), [`./competitor-landscape-synthesis.md`](./competitor-landscape-synthesis.md), [`../positioning.md`](../positioning.md), [`../specialist-network.md`](../specialist-network.md), [`../trust-gradient.md`](../trust-gradient.md).
> **Feeds into:** [`../../changes/lock-positioning-trajectory/`](../../changes/lock-positioning-trajectory/) Stage 5 fork criteria; [`./value-attribution.md`](./value-attribution.md) Q3 mechanism candidates; [`../specialist-network.md`](../specialist-network.md) § 8 (compensation, currently placeholder).
> **Path note:** Live canvas companion at `canvases/social-positioning-analysis.canvas.tsx` (workspace path). The canvas is the visual artefact; this file is the durable research record.

---

## 0. TL;DR

The conversation that opened this thread (2026-05-12) framed the question as "should the studio adopt a *social-oriented* identity?". Two parallel external research passes plus a cross-reference audit of the studio's own methodology stack produce three load-bearing findings:

1. **Across 21 analog organisations, only 8 are simultaneously strong on transparency and on equitable resource distribution.** In all 8, transparency is *structural* (legal entity, public handbook, operational tooling) — not rhetorical. "We publish salaries" is a weak signal. "Our Co-op Rules mandate patronage and here is the 2021 allocation breakdown" is a strong one. Edelman 2025 confirms the gap is closeable only by structure: 26-pt trust deficit between tech-sector and AI specifically.

2. **The studio already operates on approximately 5 of 7 trust registers** (governance documents, actionable methodology, personal/build-in-public, partial technical-primitive, partial democratic-influence via library-promotion). It does not name these externally. What is missing is (a) a chosen legal-form story, (b) a published wealth-distribution rule, (c) an external narrative connecting (a) and (b) to client-side trust. These three gaps — not "adopt social orientation as a new label" — are the real work.

3. **None of the 21 analogs operate at €50k+ engagement scale as a worker-cooperative.** The closest precedents (Outlandish, Hypha) are at smaller deal sizes. Any "studio-as-coop" path is first-of-kind at the studio's target ACV — to be treated accordingly.

Three operational tracks are proposed (Surface / Restructure / Bifurcate; § 13). They sit on a different axis than the Stage 5 fork in [`lock-positioning-trajectory`](../../changes/lock-positioning-trajectory/) (Boutique / Productized / SaaS), which is about *product shape*; this thread is about *values + distribution + trust*. The two axes are orthogonal and combinable.

---

## 1. Why this research

The 2026-05-12 strategic conversation surfaced an instinct: the studio is *socially-oriented* — it cares about people, organises specialists around domains of human concern, and positions services with a *people-first* register. The instinct is partially right, partially imprecise. "Social" in English / Hebrew / Romanian (the studio's chrome locales) decomposes into at least six incompatible registers: social impact (NGO), social enterprise (B-Corp), civic / public-interest, humane / pro-social design, people-first / relational, cooperative / solidarity. Each has a different audience and different commercial consequence.

Without analog data, picking a register is guesswork. Without internal-mapping, the studio risks either (a) adopting a label whose practice it does not match (greenwashing-class trust failure) or (b) restructuring around a register that conflicts with its existing positioning (founder-track ICP cold-pitch vocabulary constraint from [`archive/2026-05-08-add-founder-track/`](../../archive/2026-05-08-add-founder-track/) explicitly forbids `methodology` / `process` / `framework` on cold-pitch surfaces).

This research closes that gap by populating both sides — what the analog space actually looks like, and what the studio is already doing structurally — so future positioning amendments operate on evidence rather than instinct.

---

## 2. The research question (composite)

Three sub-questions, treated together because the data shows they share the same answer-space:

**Q-RES-1.** How do organisations that operate at "social / human-rights / public-interest / cooperative" level actually position themselves on five axes: legal form, network model, wealth-distribution claim, trust signal, AI maturity?

**Q-RES-2.** Where does the trust × wealth-distribution intersection live in practice? Which mechanisms (across legal, operational, contractual, narrative layers) actually move both?

**Q-RES-3.** Given the studio's existing methodology stack and commercial constraints (€50k+ B2B engagements, founder-track ICP, cold-pitch vocabulary constraint), what positioning moves are coherent — and which are vocabulary-rebrandings that would collapse on inspection?

---

## 3. Method

Two parallel research subagents:

- **Subagent A** — surveyed 21 organisations across six categories: (A) public-interest / civic AI; (B) human-rights tech; (C) mission-driven product studios; (D) platform cooperatives / worker-owned tech; (E) AI-for-good orgs; (F) European / non-US analogs. Each organisation captured on 10 axes (legal form, year/country, claim, business model, network model, wealth-distribution rhetoric, AI cases, distinguishing feature). All claims inline-cited; gaps explicitly flagged.

- **Subagent B** — surveyed trust frameworks (Edelman 2025, Mozilla Trustworthy AI, Partnership on AI ABOUT ML, Humane Tech Framework, Trust Equation, B2B trust signal research) and wealth-distribution frameworks (Platform Cooperativism / Scholz / PCC, ICA 7 Principles, worker-coop case data, profit-sharing analogues at Basecamp / Buffer / Doist / GitLab, specialist-marketplace ethical critique — Toptal / Braintrust / Working Not Working / The Dots, "AI augmentation" corporate manifestos, AI Now Institute, Cooperative AI Foundation, academic frames — Benkler / Ostrom / Mazzucato).

Cross-reference pass: matched analog data against the studio's existing methodology — `specialist-network.md`, `value-attribution.md`, `trust-gradient.md`, `positioning.md`, `red-lines.md`, the just-archived [`2026-05-12-apply-trust-research-p0/`](../../archive/2026-05-12-apply-trust-research-p0/), the in-flight [`lock-positioning-trajectory/`](../../changes/lock-positioning-trajectory/), the `contributor-taxonomy.canvas.tsx` work.

Synthesis pass: extracted seven repeating patterns, six rare differentiators, seven trust registers, eight orgs at the trust × distribution intersection, three operational tracks. All synthesis claims trace back to inline-cited primary data; where data is absent the gap is named (§ 22).

---

## 4. The 21 analog organisations

Grouped by category. Each entry: legal form · wealth-distribution claim · network model · most-distinct trust signal. Full inline-cited 10-axis data per organisation lives in the subagent transcripts; this file carries the operationally-useful subset.

### 4.A Public-interest / civic AI

| Org | Country · Year | Legal form | Network model | Distinct trust signal |
|---|---|---|---|---|
| **Code for America** | US · 2009 | 501(c)(3) non-profit | ~240 staff + civic-tech contributor base | Annual Impact Reports ($4.3B benefits delivered 2024); Anthropic partnership for SNAP Policy Navigator |
| **Nava PBC** | US · 2015 | Delaware Public Benefit Corp | 500+ W-2 employees (no contractors); **equity for all** | Annual Public Benefit Report ($3B+ payments enabled, 50M+ households) |
| **Ad Hoc** | US · 2014 | For-profit LLC (B-Corp claim unverified) | Distributed remote employees | Pragmatic civic-tech operator; weaker on trust-narrative than Nava |
| **mySociety** | UK · 2003 | Charity + SocietyWorks commercial subsidiary | Staff + opensource contributor base | **AI Framework (2024): 6-domain gating rubric — should AI be used at all** |
| **Algorithmic Justice League** | US · 2016 | Non-profit | ~2 FTE core + Agents-of-Change network | "Gender Shades"; spoken-word + peer-reviewed advocacy |
| **AI Forensics** | EU · 2021 | German e.V. + French association | Small core + open-source release of auditing tools | EC-proceedings-cited investigations; adversarial methodology |

### 4.B Human-rights tech

| Org | Country · Year | Legal form | Network model | Distinct trust signal |
|---|---|---|---|---|
| **HURIDOCS** | CH · 1982 | International NGO | Distributed staff + GitHub contributors | 40+ years methodology continuity; Uwazi open-source |
| **Benetech** | US · 2002 | 501(c)(3) | Employees + 40k+ partner schools | Bookshare (1.4M titles), software-product-org shape (rare among NGOs) |
| **Forensic Architecture + Forensis** | UK · 2010 / DE · 2021 | Research agency in Goldsmiths + Berlin e.V. | Interdisciplinary core + MA students | Counter-forensic methodology in ICJ / parliaments; Right Livelihood Award 2024 |
| **WITNESS** | US · 1992 | 501(c)(3) | 300+ partner HR-groups in 80+ countries | SHAPE-RESIST-ADOPT-REIMAGINE framework; Deepfake Rapid Response Force |

### 4.C Mission-driven product studios

| Org | Country · Year | Legal form | Network model | Distinct trust signal |
|---|---|---|---|---|
| **IDEO.org** | US · 2011 | 501(c)(3) (separate from for-profit IDEO Inc.) | Employees + Fellows + 100k-practitioner Design Kit community | Structurally-separated non-profit dock to commercial mother — rare pattern |
| **Thoughtworks Social Change Lab** | Global · ~2009 | Internal program of for-profit Thoughtworks | Full-time employees rotate into pro-bono | **"Solidarity over charity"** — sharpest anti-charity stance in the set |

> *Category C is operationally thin in reality.* IDEO.org is almost alone. Pentagram has no formalised social arm. Mathison is for-profit DEI SaaS (€25M Series A), not a social-impact studio. Reboot Studio NYC in the implied form does not surface. The "mission-driven product studio" category is a marketing label more than a working structural pattern.

### 4.D Platform cooperatives / worker-owned tech

| Org | Country · Year | Legal form | Wealth distribution mechanism (named) | Distinct trust signal |
|---|---|---|---|---|
| **Outlandish** | UK · 2010 | Worker co-operative | Quarterly surplus *proportional to hours worked*; September pilot allocations £1,542–£5,333 per person; spending via CoBudget (£192,450 across 34 projects, 2015–2018) | Public allocation numbers + CoBudget tool open-sourced |
| **Hypha Worker Co-op** | CA · 2019 | Non-share-capital non-profit worker coop (Ontario) | Wage-based; transparent 35% clip-breakdown (4% vacation + 5% holiday + CPP/EI + reserve + dev fund + solidarity) | Sells trust as *technical primitive* (verifiable computing, ZK VMs, Local AI) — only coop in set at this technical depth |
| **Common Knowledge** | UK · 2019 | Not-for-profit worker coop | Worker-ownership | Explicit prefigurative politics; explicit left positioning |
| **Loomio Co-op** | NZ · 2012 | Worker-owned coop | Equal shares; ≥40% board members must be coop members | Public Constitution; "eat-your-own-dog-food" governance |
| **Stocksy United** | CA · 2013 | BC Co-op (artists) | 50% standard / 75% extended royalty + annual patronage ($568k allocated 2021); $1 membership share | Member-owned governance + public allocation reports + Co-op Rules public |
| **Resonate** | International · 2015+ | Worker- and artist-owned cooperative | 70% to artist per stream-to-own; "track owned by listener after 9 plays" | Public stream-to-own pricing formula |
| **Up & Go** | US (NYC) · 2017 | Platform owned by 6 worker-coops | **95% to worker-cooperatives, 5% to platform**; "1 worker = 1 vote" | Sharpest single-headline distribution number in set |
| **The Drivers Cooperative** | US (NYC) · 2020 | Driver-owned cooperative | $30/hour minimum; "all profits return to drivers" (split percentage not disclosed) | 2023: $5.9M rev, 12× YoY; *but* "rise and fall" by 2024 per Documented NY |

### 4.E AI-for-good organisations

| Org | Country · Year | Legal form | Network model | Distinct trust signal |
|---|---|---|---|---|
| **DAIR Institute** | US · 2021 | Independent non-profit | Globally distributed researchers/activists/engineers | **Care-ethics for workers** in founding mission: anti-burnout, family support, "we reject academic burnout" (unique in set) |
| **Partnership on AI** | US · 2016 | 501(c)(3) | 138 partner-orgs across 20+ countries | Multi-stakeholder convening — *also* most-criticised pattern (ethics-washing risk) |
| **Data & Society** | US · 2013 | 501(c)(3) research institute | Staff + visiting fellows + extended researcher network | **Gift Acceptance Policy (2020)** — published refusal criteria; terminated Microsoft funding in 2017; funder list ≥$10k public |
| **Center for Humane Technology** | US · 2018 | 501(c)(3) | Small core + tech-insider alumni | Theory-of-change via media-narrative shift ("The Social Dilemma" mode) |

### 4.F European / non-US analogs

| Org | Country · Year | Legal form | Trajectory | Distinct trust signal |
|---|---|---|---|---|
| **Doteveryone (defunct)** | UK · 2015–2020 | Charity / think tank | Closed 2020; portfolio handed off to Ada Lovelace Institute + ODI | **Sunset as success-criterion** — only organisation in set that closed on purpose |

> Bureau Cool, Studio Folder, Mathison, Reboot Studio NYC, CoLab Co-op investigated and excluded with reason — see § 22.

---

## 5. Seven repeating positioning patterns ("default script")

Patterns that appear in ≥3 analogs. Adopting these is necessary table-stakes for any social-orientation claim; differentiation has to come from the rare patterns (§ 6).

| # | Pattern | Examples | Studio alignment |
|---|---|---|---|
| 1 | **Open-source by default as proof-of-mission** | CfA · Nava · mySociety · Benetech · HURIDOCS · AI Forensics · AJL | **Strong** — OpenSpec is structurally there; public-mirror commitment in [`apply-trust-research-p0`](../../archive/2026-05-12-apply-trust-research-p0/) D-TRP-1. Forge OSS release proposed in [`add-funding-strategy`](../../changes/add-funding-strategy/) D-FUND-5. |
| 2 | **Legal form as brand statement** | PBC (Nava); worker-coop (Outlandish/Hypha/Loomio/Common Knowledge); 501(c)(3) (CfA/DAIR/D&S/AJL/CHT); charity (mySociety); e.V. (Forensis) | **Open** — studio is SRL today. This is a decision held by [`lock-positioning-trajectory`](../../changes/lock-positioning-trajectory/) D-POS-4 (Stage 5 fork) and would re-open if a Path-B-shape choice is taken. |
| 3 | **Founder-injury / institutional-betrayal genesis story** | HealthCare.gov 2013 → Nava / Ad Hoc · Buolamwini face-rec → AJL · Gebru fired from Google → DAIR · Harris ex-Google → CHT | **Not available** — Pavel does not have an institutional-betrayal origin. Narrative needs a different shape; trust gets built through artifact (public OpenSpec) and discipline (`trust-gradient.md`), not biography. |
| 4 | **"Communities most impacted" centering** | AJL · DAIR · CfA · Thoughtworks SCL · WITNESS | **Available but cliché-risk** — needs structural backing (concrete engagement examples) or it reads as moralising. Studio has no shipped engagement to anchor this in yet. |
| 5 | **Quantified impact reporting** | "$4.3B benefits delivered" (CfA) · "$3B+ payments" (Nava) · "300+ partners in 80+ countries" (WITNESS) | **Not yet** — pipeline of engagements has to ship first. Premature to adopt this pattern at Stage 0. |
| 6 | **Foundation funding from narrow donor pool** | Ford / MacArthur / Knight / Hewlett / Open Society / Kapor — same six in 6+ donor lists | **Not applicable** to commercial B2B studio. Adjacent for any future civic-arm (Track 3 in § 13). |
| 7 | **Convening / multi-stakeholder rhetoric** | Partnership on AI · CHT · mySociety (partially) | **Avoid** — read as "AI ethics-washing" by founder-track ICP (per [`archive/2026-05-08-add-founder-track/`](../../archive/2026-05-08-add-founder-track/) cold-pitch vocabulary constraint). |

---

## 6. Six rare positioning patterns (differentiators)

Each appears in exactly 1 analog. These are the dimensions where a new entrant can plant a flag. Most overlap with what the studio already does internally but does not say publicly.

| # | Pattern | Who does it | Why it differentiates | Studio alignment |
|---|---|---|---|---|
| 1 | **Care-ethics for *own workers*, not just users** | DAIR — anti-burnout, family support, "we live full lives outside work" in founding doc | Inverts the script — most orgs say "we serve users"; DAIR says "we live" | **High** — already inside [`value-attribution.md`](./value-attribution.md) Q4 ("what does the specialist need to *see* to feel valued"). Public articulation would be small step. |
| 2 | **"Should AI be used at all" gating methodology** | mySociety AI Framework 2024 — 6-domain rubric (practical / societal / legal-ethical / reputational / infrastructural / environmental) | All others prescribe HOW; mySociety gates IF | **High** — [`trust-gradient.md`](../trust-gradient.md) RED LINE rows already gate at category level. Extension to engagement-level gating ("when do we NOT take an engagement") is a Gift-Acceptance-Policy variant. |
| 3 | **Adversarial methodology as practice** | AI Forensics — scraping, sock-puppeting, data-donation against tech platforms | Most orgs collaborate; AI Forensics confronts | **Low** — studio is B2B partner-builder. Adversarial mode is incompatible with the engagement contract. |
| 4 | **Public Gift / Engagement Acceptance Policy** | Data & Society — terminated Microsoft funding in 2017, published criteria, public funder list ≥$10k | Trust as "what we will refuse," not "what we promise" | **High** — directly portable to studio as "engagements we decline" doc. Low-cost authoring (single page) + high trust uplift. |
| 5 | **Sunset as success metric** | Doteveryone — closed 2020 with "we set the debate in motion"; portfolio handed off cleanly | Anti-scaling stance — rare in tech | **Not applicable** to commercial studio. Usable as posture for time-limited sub-products (canvas studio for a defined market window). |
| 6 | **Production-grade tech in cooperative form** | Hypha — cryptographic provenance, TEEs, Local AI in worker-coop | Most coops do "websites + dashboards"; Hypha does protocol layer | **High** — closest structural peer to studio's technical-depth claim. If Track 2 (Restructure) is ever taken, Hypha is the operating template, not Outlandish. |

---

## 7. Seven registers of trust (how orgs articulate trust)

Cross-section of the analog data. Each register is a different mechanism, not just a different vocabulary. The registers compose; an organisation can occupy multiple.

| Register | Mechanism | Examples | Studio fit |
|---|---|---|---|
| **Technical primitive** | Verifiable computing, cryptographic provenance, hand-verifiable artefacts | Hypha; partially Stocksy via Co-op Rules | Partial — OpenSpec-as-artefact has this quality; not yet named externally |
| **Governance document** | Transparency reports, gift-acceptance, donor list public | Data & Society; Partnership on AI; Wikipedia | Available — publishable as "Engagement Acceptance Policy" + open-source `decisions.md` |
| **Fiduciary duty in charter** | PBC charter, B-Corp registration, coop constitution | Nava PBC; Loomio Constitution | Decision — requires SRL restructure to PBC-equivalent or coop-equivalent; held by [`lock-positioning-trajectory`](../../changes/lock-positioning-trajectory/) D-POS-4 |
| **Democratic ownership** | Worker-coop, patronage-shares, equity-for-all | Outlandish · Hypha · Loomio · Common Knowledge · Stocksy · Nava (equity-for-all variant) | Decision — Track 2 in § 13. First-of-kind at €50k+ ACV. |
| **Investigative reputation** | Track record of evidence in court / regulator filings | Forensic Architecture · AI Forensics · WITNESS | Not applicable to studio role |
| **Actionable methodology** | Methodology that *constrains* — "what we won't do" | AJL 4 Principles · mySociety AI Framework · Data & Society Gift Policy · studio's own [`red-lines.md`](../red-lines.md) + [`trust-gradient.md`](../trust-gradient.md) | **Strong** — already in place; under-named externally |
| **Personal (Trust Equation)** | `(Credibility + Reliability + Intimacy) / Self-orientation` — built between people, not companies | Founder-led build-in-public (Basecamp · PostHog · DHH · Sahil Lavingia) | **Strong** — Pavel's public OpenSpec writing + studio handbook is exactly this |

Source-of-truth for the registers: Subagent B synthesis from Edelman 2025 + Mozilla Trustworthy AI + Trust Equation (Charles Green / Trusted Advisor) + Partnership on AI ABOUT ML + Center for Humane Technology + the analog data above.

---

## 8. Eight organisations strong on BOTH transparency AND distribution

The trust × wealth-distribution intersection. Filtered from 21 → 8 by requiring named-mechanism on both axes.

| Org | Wealth distribution mechanism | Transparency mechanism |
|---|---|---|
| **Stocksy United** | 50–75% royalty + annual patronage ($568k in 2021); $1 membership share | Public Co-op Rules; named patronage allocations; member-owned governance |
| **Resonate** | 70% to artist per stream; worker- and artist-owned | Public stream-to-own pricing formula |
| **Outlandish** | Quarterly surplus proportional to hours + CoBudget collective spend | £1,542–£5,333 per-person allocations publicly named; CoBudget tool open-sourced |
| **Loomio Co-op** | Equal worker-ownership shares | Public Constitution + open Cooperative Handbook |
| **Hypha Co-op** | Wage-based, transparent 35% clip-breakdown | Open Handbook (H₂O) + Open Collective books |
| **Nava PBC** | Equity for all employees regardless of role | Annual Public Benefit Reports; PBC charter |
| **Up & Go** | 95% to worker-coops, 5% to platform; "1 worker = 1 vote" | Public model description, open ownership structure |
| **Buffer (non-coop)** | COL × role × experience formula; 25% YoY profit-share | All salaries fully public since 2013; full formula public |

**Single load-bearing observation across the 8.** *Transparency is structural, not rhetorical.* It is anchored in legal entity (co-op rules, PBC charter), public documents (handbook), or operational tooling (CoBudget, public allocations). "We publish salaries" is a weak signal in this dataset. "Our Co-op Rules mandate patronage and here is the 2021 allocation in dollars" is a strong one.

Edelman 2025 confirms the gap is closeable *only* by structure: 26-point trust deficit between tech-sector and AI specifically; AI trust 32% in US vs 72% in China; globally only 44% comfortable with businesses using AI. Marketing language alone does not move this. Structure does.

---

## 9. Academic frameworks (theoretical fundament)

Three frames recur in the literature for any serious treatment of network-economy distribution. The studio is encouraged to read at least one before committing to Track 2 (§ 13).

### 9.1 Yochai Benkler — *The Wealth of Networks* (2006)

**Concept:** commons-based peer production — production by thousands of contributors without markets or hierarchies (Linux, Apache, Wikipedia, SETI@Home, Project Gutenberg).

**Three organisational features that make peer-production work:**
1. Decentralisation of conception and execution.
2. **Diverse motivations** — not only money.
3. **Separation of governance/management from property/contract.**

**Direct implication for studio.** The specialist network can be a "networked information economy" where specialists participate by diverse motivations (income + reputation + library-promotion + care-domain expression), governance is separated from ownership (specialists shape methodology without owning the SRL), and the default is open commons (OpenSpec as the medium). This is the **direct theoretical basis** for the existing curated-network model — not an external framework that needs to be adopted, but recognition of the frame the studio already operates in.

### 9.2 Elinor Ostrom — *Governing the Commons* (1990), 8 design principles for managed commons

1. Clearly defined boundaries (resource + community).
2. Congruence — rules match local conditions.
3. Collective-choice arrangements — participation in rule-making.
4. Monitoring — community monitors both resource and users.
5. Graduated sanctions.
6. Conflict-resolution mechanisms — fast, accessible.
7. Recognition of rights to organise.
8. Nested enterprises — polycentric governance.

**Direct implication for studio.** Treating the specialist network as a *commons*: clear membership boundaries (NDA + certification per [`specialist-network.md § 2`](../specialist-network.md)), prescribed contribution/reward rules (promotion-to-library § 6.3, compensation tiers § 4.3), peer monitoring (senior-review delta + client revisions § 7.1), low-cost conflict resolution (i-avocat counsel + studio mediation), nested structure (internal core + extended network). **All eight principles map to existing studio practice.** Naming Ostrom in the methodology — not adopting her — closes the theoretical loop.

### 9.3 Mariana Mazzucato — value extraction vs value creation (UCL IIPP)

Differentiates "network monopoly rents" + "algorithmic rents" — two mechanisms platforms (Amazon / Google / Meta / Apple) use to *extract* value rather than *create* it. Notes that internet, GPS, foundational R&D were taxpayer-funded yet value flows to private capital.

**Direct implication for studio.** Studio can articulate *value creation vs extraction* as positioning: deliverables are owned by client (`spec is in your repo from day one`), accumulated learning compounds in shared library not in vendor-trap, network participants are named not anonymised. This is a Pillar-B-aligned narrative already partly carried in `archive/2026-05-06-expand-home-with-services-and-community/research/muse-trust-and-ownership.md`.

---

## 10. Trust × wealth-distribution intersection — what the synthesis says

Seven practical principles emerge from the analog data + framework synthesis. Each is anchored in the source data above; this section is the operational digest.

| # | Principle | Source anchor | Studio status |
|---|---|---|---|
| 1 | **Trust is built structurally, not rhetorically.** Public claims of "transparency" / "fairness" are weak without legal/operational structure behind them. | Edelman 2025; § 8 above (all 8 org pattern) | Partial — `decisions.md` + `trust-gradient.md` + `red-lines.md` are structural; not yet externally indexed |
| 2 | **Curated network ≠ pyramid; specify how value flows.** Toptal lost trust through ~50% opaque margin + 24-mo non-circumvent. Outlandish (proportional-to-hours) and Stocksy (50–75% royalty + patronage) publish the formula. | Subagent B § 4 (Toptal critique); analog data | Partial — `specialist-network.md § 6.3` (library-promotion + named royalty) exists; not yet quantified or published |
| 3 | **AI as augmentation, articulated structurally.** Corporate "augmentation not replacement" manifestos (IBM / Salesforce / Asana / Paramount) are everywhere; structural guarantees (contracts, equity-share, accountable human-in-loop) are nowhere. | Subagent B § 5; SEC AI-washing enforcement | Partial — `trust-gradient.md` RED LINE rows + Forge UI human-merge gate are exactly the structural form. Public articulation pending. |
| 4 | **Open methodology > open marketing.** Open-sourcing audit-engine / scoring framework / delivery methodology (AI NYC; nao; PostHog) creates a structural moat ("individually copyable, collectively hard to replicate"). | Agency Forward research | Strong — OpenSpec public mirror committed via `apply-trust-research-p0` D-TRP-1 |
| 5 | **Data sovereignty as offensive, not defensive.** EU B2B buyers require *three* layers of proof — contractual (DPAs, residency, subprocessor lists) + technical (customer-controlled keys in their jurisdiction, single-tenant, geofencing) + operational (audit logs, access records, incident reports). Decisive factor: where the encryption keys live. | Kiteworks / IterationLayer research | Partial — studio's "spec in your repo, every commit yours" satisfies operational layer; contractual layer needs explicit articulation in engagement docs |
| 6 | **Public post-mortems and public pricing are asymmetric trust signals.** Industry where 80% B2B agencies do not publish pricing and incident post-mortems are rare → publishing both yields disproportionate uplift. Atlassian 2017: +12% on stock after open breach disclosure. | UpliftGTM research; Atlassian case | Pending Stage 1 — pricing floor (Q-POS-2) and engagement post-mortems are in `lock-positioning-trajectory` scope |
| 7 | **Equity or patronage for core network, not just founders.** Hybrid "looks-like-coop-but-without-equity" is detected and dismissed by market (Braintrust DAO-overlay critique). Three structural options: patronage (Stocksy), worker-owner core + flexible network (Outlandish), PBC universal equity (Nava). **Pick one; do not blend.** | Subagent B § 1–2; Braintrust critique | Open — see Track 2 in § 13. Held by [`lock-positioning-trajectory`](../../changes/lock-positioning-trajectory/) D-POS-4. |

---

## 11. What the studio already has (existing values architecture)

Before adopting external positioning, inventory the internal. The following internal artefacts already populate ~5 of 7 trust registers (§ 7) — they are not named externally as such.

| Artefact | Trust register it carries | Wealth-distribution claim it carries |
|---|---|---|
| [`specialist-network.md § 0`](../specialist-network.md) (thesis) | Actionable methodology + governance document | Implied: network as compounding asset |
| [`specialist-network.md § 6.3`](../specialist-network.md) (promotion-to-library) | Governance document | **Explicit patronage mechanism** — named contributor in metadata + compensation tier step-up across ≥3 engagements |
| [`specialist-network.md § 3`](../specialist-network.md) (NDA model) | Actionable methodology (protective layer) | Implied: studio absorbs NDA complexity for specialist |
| [`value-attribution.md`](./value-attribution.md) | Governance document (research-thread shape) | **Open mechanism candidates** for Q3 value-concentration prevention |
| [`trust-gradient.md`](../trust-gradient.md) + [`red-lines.md`](../red-lines.md) | Actionable methodology — RED LINE rows | N/A — technical trust |
| Pavel's public OpenSpec writing + change-folder discipline | Personal (Trust Equation) | N/A |
| OpenSpec public-mirror commitment via [`apply-trust-research-p0`](../../archive/2026-05-12-apply-trust-research-p0/) D-TRP-1 | Technical primitive + open-source-default | N/A |
| `contributor-taxonomy.canvas.tsx` (4 tiers, 55+ roles) | Governance document | Implied: "every specialty has a place" map |
| [`decisions.md`](../../decisions.md) | Governance document | N/A |

**Synthesis.** The studio operates on roughly 5 of 7 trust registers already (governance document, actionable methodology, personal/build-in-public, partial technical-primitive, partial democratic-influence-via-library-promotion). What is missing: (a) a chosen legal-form story, (b) a published wealth-distribution rule, (c) an external narrative connecting (a) and (b) to client-side trust.

These three gaps are the real work — not "adopt social orientation as a new label". The label without structure is detected and dismissed (see § 6 row 4 — D&S vs orgs that claim independence without published gift policy).

---

## 12. Specialist categorisation — three models on the network surface

The 2026-05-12 conversation surfaced an instinct: organise specialists by *social sphere they help with*. The instinct has substance, but the operational implementation has three shapes. Below is the trade-space.

| Model | Primary axis | Secondary axis | Pros | Cons | Best for |
|---|---|---|---|---|---|
| **A. Craft-only (status quo)** | Discipline (illustrator, frontend, strategist, ...) | None | Industry-standard; clients know what skill to hire | Generic; does not differentiate from any agency | Default if no positioning shift |
| **B. Sphere-first (radical)** | Social domain (healthcare, civic, education, climate, ...) | Craft | Distinctive; aligns with social-impact register | Forces taxonomy where domain expertise is thin; clients cannot filter by skill | Pure-social-impact studio (e.g. IDEO.org clone) — *wrong fit* for €50k+ B2B |
| **C. Craft × care-domain (hybrid)** | Discipline (existing 4-tier `contributor-taxonomy` axis) | **Care domain** — optional, self-declared, evidence-gated (engagements + NDAs + publications) | Preserves craft fit; adds values dimension; honest about specialists who have no care-domain (they keep status quo) | Two-axis filter UX is harder than one | **Recommended.** Fits existing taxonomy, opens new narrative, low-cost. |

**Honest caveat on Model B.** A frontend specialist with 8 years in healthcare is meaningful. A frontend specialist tagged "healthcare" because the studio wanted to look social is performative — clients smell it. Model C handles this: care-domain is optional, self-declared, requires evidence (engagements / NDAs / publications), rendered as *additional* context — not replacement for craft.

**Implementation cost (Model C).** Schema field on specialist profile (`care_domain: string[]` — small enum, expandable); filter UI on `/network`; one paragraph in `specialist-network.md § 6` defining evidence-gating; review at first three certifications to validate the evidence-bar. Total: small change folder, ~1 sprint.

---

## 13. Three operational tracks (Surface / Restructure / Bifurcate)

Three internally-coherent paths the studio can take. **Do not blend.**

> **Terminology note.** These tracks are intentionally renamed from "Path A / B / C" to avoid collision with the Stage 5 fork in [`lock-positioning-trajectory`](../../changes/lock-positioning-trajectory/) (Boutique / Productized / SaaS). The Stage 5 fork is about *product shape*; this thread is about *values + distribution + trust*. The two axes are orthogonal — Track 1 (Surface) combines naturally with any Stage 5 path; Track 2 (Restructure) restricts the Stage 5 fork meaningfully; Track 3 (Bifurcate) opens a parallel venture surface.

### 13.1 Track 1 — Surface (evolutionary; recommended starting move)

**Posture.** Studio remains commercial B2B engagement-builder whose practice is values-driven. Do *not* call it "social". Make existing values mechanics visible.

**Moves.**

1. Publish studio handbook publicly (Basecamp / GitLab template), including `specialist-network.md`, `value-attribution.md`, `trust-gradient.md`, `red-lines.md` — already drafted, just exposed. Public mirror already committed via [`apply-trust-research-p0`](../../archive/2026-05-12-apply-trust-research-p0/) D-TRP-1.
2. Adopt Model C (craft × care-domain) on `/network` — small change folder.
3. Publish Engagement Acceptance Policy (Data & Society template) — "engagements we decline, and why". Single page; founder-authored; published next to `decisions.md`.
4. Convert `value-attribution.md § Q3` answers into a published wealth-distribution rule (Stocksy patronage shape: % to specialist · % to studio · % to library-promotion royalty pool). The mechanism is already drafted in `specialist-network.md § 6.3`; what is missing is the *quantification* and the *public form*.
5. Keep cold-pitch surface clean of social vocabulary (`add-founder-track` constraint remains intact). The values articulation lives on `/services` deep-dive pages, `/lab`, the handbook — not on Hero or `/founders`.

**Closest analog.** Hypha Worker Co-op (without the legal-form change) + Data & Society governance docs.

**Risk.** Low. Existing positioning preserved.

**Upside.** Medium — "build-in-public" trust premium with founders; closes 4 of 7 trust registers structurally (open-source default + governance document + actionable methodology + personal); meaningful but not transformative.

**Reversibility.** Fully reversible. Pages can be unpublished; documents revert to internal.

**Time.** 2–3 months.

**Compatible with Stage 5 fork.** All three Stage 5 paths (Boutique / Productized / SaaS) absorb Track 1 without conflict.

### 13.2 Track 2 — Restructure (radical; high-conviction only)

**Posture.** Studio is legally restructured into a worker-cooperative core + flexible certified-specialist network. Wealth-distribution becomes the headline.

**Moves.**

1. Investigate Moldovan / Romanian / EU cooperative legal forms with i-avocat. **Critical gating constraint** — no live MD/RO worker-coop framework matches UK / Canadian / NZ models at €50k+ ACV. Decision: either find a structure that fits, or this track does not exist.
2. Core team (Pavel + 5–10 onboarded specialists) become member-owners (Outlandish model). Extended specialists participate via per-engagement contract + library-promotion patronage.
3. Publish constitution. Adopt CoTech / ICA 7 Principles framework.
4. Pricing changes: surplus distributed quarterly by hours-worked formula (Outlandish CoBudget template) or contribution-weighted formula (Stocksy patronage template). Pick one; publish.

**Closest analog.** Outlandish + Hypha.

**Risk.** High.
- Conflicts with founder-track ICP (Israeli founders read "coop" as "slow / committee-driven", per [`competitor-landscape-israel.md`](./competitor-landscape-israel.md) ethnographic findings).
- Restructuring cost (legal + organisational) is non-trivial.
- Pavel's sole-ownership of studio IP becomes negotiated; ratchet decision.

**Upside.** Very high — unique positioning at the AI + B2B + EU intersection. No live competitor with this exact shape. Major-media-class story.

**Reversibility.** Very low. Once equity is distributed, undoing it is a buy-out event.

**Reality check.** None of the 21 analogs operate at €50k+ engagement scale as a worker-coop. The closest precedent (Outlandish, Hypha) is at smaller deal sizes. This path is **first-of-kind at the studio's target ACV** — treat as such.

**Time.** 6–12 months (gated on legal feasibility study).

**Compatible with Stage 5 fork.** Track 2 restricts the Stage 5 fork meaningfully. Path B (Productized methodology) is most compatible (cooperative + methodology product). Path C (SaaS) is hard — coop investors are rare. Path A (Boutique) is possible but loses some commercial flexibility.

### 13.3 Track 3 — Bifurcate (split-track; IDEO pattern)

**Posture.** Commercial studio stays as-is (€50k+, founder-track, MD/RO + IL). A second arm — sub-brand, separate legal entity, or program — opens for pro-bono / discounted engagements with NGOs, government, public-interest projects.

**Moves.**

1. Open `/civic` or `/lab/social` page on `apps/web` — explicit pro-bono / discounted pipeline.
2. Adopt Thoughtworks SCL "solidarity over charity" framing (avoid pure-NGO register).
3. Specialists with personal civic agenda get visibility through civic-arm engagements — voluntary, declared.
4. Anchor first 1–2 civic-arm engagements (Moldovan govtech pilot; EU-funded civic project; HURIDOCS-style human-rights tooling) *before* announcing the arm publicly. Otherwise the announcement is decoration.

**Closest analog.** IDEO → IDEO.org · Thoughtworks → Social Change Lab · Pentagram's partner-led pro-bono pattern.

**Risk.** Medium.
- Operational overhead — two go-to-market motions, two contract templates, two pricing tables.
- Resource dilution — if civic-arm draws core specialists, paying clients notice.
- Greenwashing detection — if civic-arm is decorative, the gap is detected fast.

**Upside.** Medium-high — credible "social presence" without compromising commercial positioning. Founder-track and Client-track stay clean. Specialists with care-domain motivations have an internal home.

**Reversibility.** Medium. Closing a sub-brand is operationally cheap if no shared infrastructure; high if shared.

**Time.** 3–6 months.

**Compatible with Stage 5 fork.** Track 3 absorbs into any Stage 5 path. Path A (Boutique) keeps civic-arm as a Pavel-time-allocation; Path B (Productized) might productize a civic-grade variant; Path C (SaaS) likely shuts the civic-arm (capital pressure).

**Adjacent venture overlap.** [`onboard-ai-commons-moldova`](../../changes/onboard-ai-commons-moldova/) is the studio's first concrete adjacent venture, governed by [`related-ventures.md`](../related-ventures.md). Track 3's civic-arm pattern may or may not align with how AI Commons Moldova is structured — needs a cross-read before commitment.

### 13.4 Combination matrix

| | Track 1 (Surface) | Track 2 (Restructure) | Track 3 (Bifurcate) |
|---|---|---|---|
| Stage 5 = Path A (Boutique) | natural fit | possible — restricts revenue flexibility | natural fit |
| Stage 5 = Path B (Productized) | natural fit | **best fit** — coop + methodology product | possible |
| Stage 5 = Path C (SaaS) | natural fit | hard — coop investors rare | hard — capital pressure shuts civic arm |
| Track 1 + Track 3 | most common adoption shape | — | — |
| Track 1 + Track 2 | — | restructure is the bigger move | — |
| All three simultaneously | not recommended — fragmenting | — | — |

**Default recommendation.** Track 1 starts. Pavel decides between Track 2 and Track 3 at Stage 4 review, informed by Stage 1–4 evidence (revenue mix, network shape, founder-track-vs-civic-demand ratio).

---

## 14. Methodology candidates worth adopting or reading

Concrete artefacts from the analog set, ranked by fit and adoption cost.

| Framework | Source | What it gives the studio | Adoption cost |
|---|---|---|---|
| **D&S Gift Acceptance Policy** | datasociety.net (2020) | Template for "engagements we decline" doc — actionable trust | Low — 1 page, founder-authored |
| **mySociety AI Framework** | research.mysociety.org (2024) | 6-domain "should we use AI" gating rubric — pairs with `trust-gradient.md` | Low — adapt to engagement-acceptance check |
| **Trust Equation (Charles Green / Trusted Advisor)** | `(Credibility + Reliability + Intimacy) / Self-orientation` | Diagnostic for every public-facing claim — does it raise or lower self-orientation? | Zero — mental model |
| **Mozilla Trustworthy AI white paper** | mozillafoundation.org (2020) | 5 principles (Agency / Accountability / Privacy / Fairness / Safety) | Low — language pack for AI-trust copy |
| **Public Interest Technology framework** | Ford Foundation + Schneier (RSA 2019) | Vocabulary for civic-arm (Track 3) | Low — naming framework |
| **7 Cooperative Principles (ICA)** | International Co-operative Alliance | Constitutional language if Track 2 chosen | High — implies Track 2 legal restructure |
| **Benkler — peer production triad** | *Wealth of Networks* (2006) | Theoretical basis for "diverse motivations + governance ≠ ownership" | Zero — citation only |
| **Ostrom — 8 commons design principles** | *Governing the Commons* (1990) | Operational principles for the specialist commons — clear boundaries, peer monitoring, polycentric governance | Low — direct fit to network-design questions; all 8 already implicitly present |
| **The Care Manifesto** | The Care Collective (2020) | Strict definition of "care" that does not collapse to softness | Low — language pack for internal posture |
| **CoTech Constitution** | wiki.coops.tech | Concrete coop network governance template | Medium — relevant only under Track 2 |

---

## 15. AI-trust market context (the data that frames any positioning move)

Five data points that frame any AI-mediated studio's trust claim. All from public sources.

1. **Edelman Trust Barometer 2025 (Tech Sector).** Global trust in tech: 76% (+2 pp over 10y); but US: 73% → 63%. Tech-vs-AI gap: **26 points**. US AI trust: 32%. Globally only 44% comfortable with businesses using AI.

2. **SEC AI-washing enforcement (2024–2025).** Three concrete fines on US public companies for false AI claims — Delphia ($225k), Global Predictions ($175k), Presto Automation (January 2025). CETU declared AI-washing "immediate priority" May 2025. Focus: "companies simply repackaging rule-based automation under an 'AI' label."

3. **B2B trust signals — Capability / Character / Reputation triad.** 80% of B2B deals won by "pre-contact favorite". 70% of buying journey happens before first contact. First impression formed in 50ms.

4. **Open methodology as structural moat.** Cited finding: "individually copyable but collectively hard to replicate." Open-sourcing the audit-engine / scoring framework / delivery methodology produces (a) trust via audit-ability, (b) lead-generation via discoverability, (c) recruitment-filter.

5. **EU data sovereignty as competitive lever.** 87% of IT/analytics leaders prioritise data management; 8 in 10 customers willing to switch on data-management doubt. EU buyers require *three* layers of proof — contractual, technical, operational. Decisive factor: where the encryption keys live.

**Implication for studio.** Any AI-orchestration claim the studio publishes must be auditable. OpenSpec-as-artefact + open prompts/tools registry (per `specialist-network.md § 6`) + the published-mirror commitment (per `apply-trust-research-p0` D-TRP-1) is the studio's already-built defence. Articulating this defence externally is the Track 1 (Surface) move.

---

## 16. Open questions

| ID | Priority | Status | Question | Blocks |
|---|---|---|---|---|
| Q-SOC-1 | high | open | Is the goal external positioning (marketing) or internal structure (legal/operational)? | Track choice; Pavel-only decision |
| Q-SOC-2 | high | open | Does Moldovan / Romanian / EU law support a worker-coop legal form at studio ACV (€50k+ engagements)? | Gates Track 2 entirely; owned by i-avocat per [`i-avocat.yaml`](../projects/i-avocat.yaml) |
| Q-SOC-3 | medium | open | Does Pavel's separate trust-research thread (referenced in 2026-05-12 conversation) overlap with § 7 of this file? Merge if so. | Determines whether trust-vocabulary in § 7 is final or interim |
| Q-SOC-4 | medium | open | Should "care-domain" tagging on specialists (Model C in § 12) be self-declared, evidence-gated, or both? | Determines whether Model C is performative or substantive |
| Q-SOC-5 | high | open | Target funnel for Track 3 (civic-arm) — Moldovan govtech / EU-funded civic / international human-rights NGOs / parallel? | Track 3 first-engagement choice |
| Q-SOC-6 | high | open | Is the "every specialty earns" thesis → patronage formula (Stocksy-style) or → core/extended-network split (Outlandish-style)? Both are valid; one must be picked. | [`value-attribution.md`](./value-attribution.md) Q3 → mechanism candidate; consumed by [`specialist-network.md § 8`](../specialist-network.md) when graduated |
| Q-SOC-7 | medium | open | Where does this research's *operational* output (Track 1 actions specifically — handbook publication, engagement acceptance policy, wealth-distribution publication) land in the change-folder graph? Absorb into [`lock-positioning-trajectory`](../../changes/lock-positioning-trajectory/) Phase 1? New sibling change? Per-action atomic changes? | Implementation routing |

---

## 17. Relationship to other studio documents

| Document | Relationship |
|---|---|
| [`./value-attribution.md`](./value-attribution.md) | This file is the **evidence base for Q3** (value-concentration prevention). Mechanism candidates (engagement-diversity floor, library-promotion royalty, new-discipline incubation budget, founder-time obligation) are explored here with analog data. |
| [`../specialist-network.md`](../specialist-network.md) | This file is the **evidence base for § 8 (compensation)** when that section graduates from placeholder to spec. Patronage-style / worker-owner core / PBC-style — three structural options grounded in analog data. |
| [`../trust-gradient.md`](../trust-gradient.md) | Operational trust (technical AI-accuracy gradient). This file is **adjacent** — relational/governance trust. The two compose: technical trust + governance trust = full external-trust posture. |
| [`../positioning.md`](../positioning.md) | The Canvas Studio thesis (hypothesis under validation). This file is **input substrate** for the validation experiment — particularly the section asking what category we play in. The "social-orientation" framing tested here is a candidate sub-thesis. |
| [`../red-lines.md`](../red-lines.md) | Path-scoped red lines. This file's "Engagement Acceptance Policy" idea (§ 6 row 4 + Track 1 step 3) is a **policy-scoped red line** — adjacent but distinct. The path-vs-policy scope question is held by [`add-funding-strategy`](../../changes/add-funding-strategy/) § Conflicts surfaced item 1. |
| [`../related-ventures.md`](../related-ventures.md) | This file is **adjacent** to related-ventures. Track 3 (Bifurcate) intersects with how the studio relates to civic-arm ventures (related as wholly-owned sub-brand, or as related-venture per the policy in this doc, or as separate legal entity). Cross-read needed before Track 3 commitment. |
| [`../../changes/lock-positioning-trajectory/`](../../changes/lock-positioning-trajectory/) | This file is the **evidence base for Stage 5 fork indicator sets** (§ 3.1–3.3 of `design.md`). The three Stage 5 paths (Boutique / Productized / SaaS) are orthogonal to the three Tracks in § 13. Both axes apply. |
| [`../../changes/add-funding-strategy/`](../../changes/add-funding-strategy/) | This file is **adjacent**. Track 2 (Restructure) interacts with funding posture — a coop cannot easily take equity capital; pricing floor (Q-POS-2) reads funding-posture as input. |
| [`../../archive/2026-05-12-apply-trust-research-p0/`](../../archive/2026-05-12-apply-trust-research-p0/) | Recently archived. The public-openspec commit (D-TRP-1), engagement closure protocol (D-TRP-2), and Pillar B mode-caveat (D-TRP-3) are foundational moves that **Track 1 of this file builds on**. Track 1 is the next layer of the same posture. |
| [`./competitor-landscape-synthesis.md`](./competitor-landscape-synthesis.md) | This file is **complementary**. Competitor landscape covers *who else builds what we build*; this file covers *who else takes positions like we might take*. Different axes; both feed positioning. |
| [`./open-source-positioning.md`](./open-source-positioning.md) | This file is **adjacent**. OSS-positioning research explores publication-vehicle naming for new professions; this file explores the broader structural-positioning landscape. May converge in a later synthesis pass. |

---

## 18. What this thread is *not*

Mirrors the discipline in [`value-attribution.md § Things this thread is not`](./value-attribution.md):

- It is **not** a positioning proposal. [`lock-positioning-trajectory`](../../changes/lock-positioning-trajectory/) is the place where positioning commitments live; this file is upstream evidence.
- It is **not** a methodology decree. The three tracks (§ 13) are options, not recommendations; the default is Track 1 because it is reversible, not because it is correct.
- It is **not** a legal-form proposal. Track 2's viability is gated on a real legal-feasibility study by i-avocat — without that study, the track does not exist as a live option.
- It is **not** founder-philosophy in isolation. Anything this thread concludes needs validation against actual specialist behavior and actual client response before it becomes policy.
- It is **not** a renaming exercise. The studio is not being asked to adopt "social-oriented" as a label. The thread asks what *structural* moves are coherent with the existing values architecture.

---

## 19. How this thread closes

Either:

1. The questions get answered enough that a sequence of small `proposal.md` change folders can be drafted: (a) `publish-studio-handbook`, (b) `add-engagement-acceptance-policy`, (c) `publish-wealth-distribution-rule`, (d) `add-care-domain-axis` — *all Track 1 actions*. Tracks 2 and 3 close separately, each via a dedicated proposal that absorbs i-avocat findings (Track 2) or first-engagement evidence (Track 3).
2. The questions resolve into "Stage 1 ratification absorbed Track 1 implicitly; Tracks 2 and 3 are S4-decisions" — this file becomes a watch-item with explicit re-open triggers tied to Stage 4 fork-review.

Either way, **no commitments are made from this file**. It is a research thread, not a spec.

---

## 20. Adjacent prior thinking

| Source | Relationship |
|---|---|
| [`./value-attribution.md`](./value-attribution.md) | Q3 (value concentration prevention) is the operational shadow of § 10 + § 13 here. Mechanism candidates trace back to analog data. |
| [`../specialist-network.md § 0`](../specialist-network.md) | The multiplier thesis (OpenSpec + Muse + contributed prompts) is the studio's home-grown version of Benkler's peer-production triad (§ 9.1). |
| [`../specialist-network.md § 6.3`](../specialist-network.md) | Library-promotion + royalty mechanism is a Stocksy-class patronage mechanism — already present, awaiting public naming. |
| [`../specialist-network.md § 8`](../specialist-network.md) | Compensation placeholder. This file is its evidence base. |
| [`./competitor-landscape-synthesis.md § 3.D`](./competitor-landscape-synthesis.md) | Israeli founder ICP read ("referral-or-nothing"). Constrains Track 2's commercial viability — Israeli cold-pitch and coop-posture are vocabulary-incompatible. |
| [`./open-source-positioning.md`](./open-source-positioning.md) | Naming dimension for new professions. Adjacent to § 14 methodology-candidate adoption. |
| [`../../archive/2026-05-08-add-founder-track/`](../../archive/2026-05-08-add-founder-track/) | Cold-pitch vocabulary constraint. Track 1 explicitly respects this (no "social" / "methodology" / "process" / "framework" on Hero or `/founders`). |
| [`../../archive/2026-05-12-apply-trust-research-p0/`](../../archive/2026-05-12-apply-trust-research-p0/) | The four-agent trust-research package. Pillars and trust-signal mapping in that package overlap with § 7 here — Q-SOC-3 holds the merge decision. |

---

## 21. Honest caveats and data gaps

- **None of the 21 analogs operate at €50k+ engagement scale as a worker-coop.** Outlandish, Hypha at smaller ACV. Track 2 is first-of-kind at the studio's target deal size — to be treated as such.
- **Scholz's full 10 platform-cooperativism principles** not all in open reprints (8 confirmed). Original on page 18 of 2016 Rosa Luxemburg Stiftung PDF.
- **Bureau Cool, Studio Folder, Mathison, Reboot Studio NYC, CoLab Co-op** — investigated and excluded with reason: Bureau Cool / Studio Folder are creative-aesthetic studios without social-impact mission rhetoric; Mathison is for-profit DEI SaaS ($25M Series A); Reboot Studio NYC in the implied form does not surface; CoLab Co-op not confirmed (results point to VC-backed CoLab Software).
- **Ad Hoc legal form** (LLC vs PBC vs B-Corp) unverified — one source claims B-Corp, not confirmed in B Lab registry.
- **"Augmentation, not replacement" corporate manifestos** (IBM, Salesforce, Asana, Paramount) are rhetorically abundant. Structural / contractual / equity guarantees of the same were **not found** in public sources. Studio's promotion-to-library royalty mechanism is structurally adjacent — public binding articulation would be a real version of this manifesto class.
- **Drivers Cooperative revenue/profit-share numbers** — $30/hr minimum confirmed; split percentage not publicly disclosed; reported "rise and fall" by 2024.
- **Design-studio compensation transparency** — outside Buffer / Basecamp / Doist / GitLab, public data is thin. The market itself is opaque; the data gap is itself a finding.
- **Cooperative AI Foundation** — grant body, not operational framework for studios. Not directly applicable to studio operations; useful as research-funder for Track 3 civic-arm.
- **The user-mentioned separate trust-research thread** — referenced in the 2026-05-12 conversation but not located by file path. Q-SOC-3 holds the merge decision.

---

## 22. Change log

| Date | Author | Change |
|---|---|---|
| 2026-05-12 | Pavel (founder) + Claude Opus 4.7 | Opened thread. Captured 21-org dataset, 7+6 patterns, 7 trust registers, 8 high-aligned orgs at intersection, 3 academic frames, 7 synthesis principles, 5-of-7 internal-trust-register inventory, 3 specialist categorisation models, 3 operational tracks (Surface / Restructure / Bifurcate — renamed from initial canvas Path A/B/C to avoid Stage 5 fork collision in `lock-positioning-trajectory`), 10 methodology candidates, 5 AI-trust market-context data points, 7 open questions Q-SOC-1..7. No commitments. |
