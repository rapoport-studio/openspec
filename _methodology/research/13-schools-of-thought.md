# 13 schools of thought around digital platforms + AI infrastructure

> **Status:** complete (snapshot, 2026-05-11)
> **Owner:** Pavel Rapoport (founder).
> **Authors:** Pavel Rapoport <pavel@rapoport.studio> (founder, primary); Claude Opus 4.7 (Anthropic) — broad-map vocabulary research across 10 disciplines (Engineering/SRE, Security, QA, BA, Ops/Legal/Finance/Sales, Brand/Marketing, Research/Discovery, Education, AI Engineering, AI Safety) + 3 emerging (Service Design, AI Engineering, AI Safety/Red Team), industry references (Google SRE, Will Larson, Shostack, BABOK, MEDDIC, Cagan, Erika Hall, Stickdorn, Diataxis, Hamel Husain, Anthropic RSP, MITRE ATLAS), ~36 search queries.
> **Method:** Single-pass synthesis. Sources include canonical books and blogs of each discipline (Google SRE Workbook, Adam Shostack's threat-modeling guide, IIBA BABOK Guide, Marty Cagan SVPG, Erika Hall, Tom Johnson's "I'd Rather Be Writing", Anthropic engineering blog, Hamel Husain evals series). For each school: roles, vocabulary (15-25 terms), artifacts, mapping to Rapoport's OpenSpec, criticality rating.
> **Reusable for:** `bridge-vocabulary-and-restructure` (Epic 1) — bridge glossary collision matrix; `expand-onboarding-cheat-sheets` (Epic 6) — content for 13 role-specific one-pagers; `establish-risk-framework` (Epic 3) — multi-school risk taxonomy. Authorship: open.
> **Opened:** 2026-05-11
> **Feeds into:** Epic 1 (collision matrix in glossary), Epic 3 (risk register multi-school), Epic 6 (lazy onboarding cheat sheets).

---

## TL;DR

Product (PM/Designer/Architect) is one school out of 13 that participate in creating digital platforms + AI infrastructure. Each school has 15-25 daily-use terms, distinct artifacts, and methodologies. **10 word collisions** matter most because AI agents hallucinate on undefined cross-school terms (`capability`, `eval`, `persona`, `voice`, `journey`, `contract`, `postmortem`, `golden dataset`, `risk`, `system card vs blueprint`). Top-5 critical schools for Rapoport Studio: AI Engineering, Engineering/SRE, Security, Ops/Legal/Finance/Sales, Education. Top-3 forgotten: Business Analysis, AI Safety, Research & Discovery.

---

## The 13 schools — at a glance

| # | School | Roles | Top artifact | Criticality |
|---|---|---|---|---|
| 1 | Product *(already mapped)* | PM, Designer, Architect | PRD, user story, roadmap, OKR | HIGH |
| 2 | Engineering / SRE | Dev (Junior→Principal), DevOps, SRE, Platform Eng, ML Eng, Data Eng | Runbook, postmortem, ADR, SLO, error budget, DORA report | HIGH |
| 3 | Security & Risk | AppSec, Threat modeler, Compliance, Red team | Threat model (Shostack 4Q), risk register, SOC2 control matrix | HIGH |
| 4 | Quality | SDET, Quality Eng, Reliability Eng | Test charter (SBTM), contract test suite, mutation testing | MEDIUM-HIGH |
| 5 | Business Analysis | BA, Solutions Consultant, Process analyst | BRD, BPMN, value stream map, RACI | MEDIUM |
| 6 | Ops / Legal / Finance / Sales | COO, Legal, CFO, Sales/AM | MSA, SOW, NDA, DPA, MEDDIC scorecard, runway model | HIGH |
| 7 | Brand & Marketing | Brand strategist, Content strategist, Copywriter | Positioning doc (Dunford), brand book, voice & tone matrix | MEDIUM |
| 8 | Research & Discovery | UX researcher, Ethnographer, Behavioral scientist, Service designer | JTBD, customer journey, service blueprint, empathy map | MEDIUM-HIGH |
| 9 | Education & Knowledge Transfer | Instructional designer, Tech writer, Knowledge manager | Bloom's learning objectives, ADDIE phase doc, Diataxis docs site | HIGH (growing with network) |
| 10 | AI Engineering | Prompt eng, ML eng, Evals specialist, AI PM | Prompt library, eval suite, trace dashboard, RAG eval matrix | HIGH (the product itself) |
| 11 | AI Safety / Ethics / Red Team | AI red teamer, Responsible AI lead | System card, model card, RSP mini-policy, ATLAS report | MEDIUM-HIGH (rising) |
| 12 | Service Design *(emerging)* | Service designer, Strategic designer | Service blueprint, journey map, touchpoint map | MEDIUM |
| 13 | Domain Expert *(per client)* | Lawyer, Doctor, Accountant, Teacher, Government official | Ontology, regulation map, case law, clinical protocol | HIGH per engagement |

## Top-5 critical for Rapoport Studio

1. **AI Engineering (#10)** — the product itself. Without eval-discipline, AI features degrade silently. Memory already records 4+ AI-related failure modes; formal eval suite still absent.
2. **Engineering / SRE (#2)** — without runbooks/SLO/postmortems impossible to operate platforms 24/7 for businesses/governments. Memory shows pattern (`worker_secrets_drift_from_infisical`) — runbook-shaped gap.
3. **Security & Risk (#3)** — MSA-gating function. SOC2 Type II practically required for enterprise tier. OWASP LLM Top 10 + MITRE ATLAS = industry lingua franca.
4. **Ops / Legal / Finance / Sales (#6)** — Moldovan SRL ↔ EU/US corporates: no MSA = no invoice. Pavel = solo founder → burn matters from day one. MEDDIC = 30-min investment with 10× pre-sales return.
5. **Education & Knowledge Transfer (#9)** — curated-team model = new specialists monthly. Diataxis-clean docs not luxury but **infrastructure** for network scaling.

## Top-3 forgotten schools

1. **Business Analysis (#5)** — programmers think "PM + Architect cover this". In reality BPMN / VSM / RACI are critical for corporate/government tenders. Without BRD on their template, deals don't pass procurement.
2. **AI Safety / Red Team (#11)** — seems "far-future, only for Anthropic". Actually EU AI Act + corporate due-diligence already ask for system cards even from consultancies that deploy frontier models.
3. **Research & Discovery (#8)** — substituted with "PM does customer interviews". Behavioral science (Kahneman/Thaler) and service design (Stickdorn) are separate disciplines with their own artifacts.

## Bridge Glossary — where vocabularies collide

The single most valuable finding for AI agents reading specs. One word, multiple schools, multiple meanings → AI hallucinates. Glossary must show all meanings + Rapoport canonical.

| Term | Meanings across schools | Rapoport canon |
|---|---|---|
| **Capability** | OpenSpec (spec unit) / Team Topologies (team capability) / AI Safety (capability threshold) / BA (business capability) / TOGAF (business capability) | **OpenSpec spec unit**; other meanings with prefix ("team capability", "AI capability threshold") |
| **Eval / test** | AI Engineering (probabilistic LLM-as-judge, 75–90% agreement target) / QA (deterministic suite) / Education (formative/summative) | **Eval = AI**; **Test = code**; **Assessment = learner** |
| **Persona** | UX (data-driven user) / Brand (Jungian archetype) / Security (threat actor) / Lifecycle (Rapoport agent persona) | **User persona** for UX; **Role** (replacing agent persona — Pavel decision); **Threat actor** for security; brand archetype only in brand context |
| **Voice** | Brand (constant identity) / UX writing (tone-modulated) / AI Engineering (assistant persona) | Brand voice = single constant; tone = contextual; AI agent voice = layer over brand |
| **Journey / Flow** | UX (customer journey) / BA (BPMN process) / Security (kill chain) / Sales (pipeline) / Education (learning path) | Prefix mandatory; never just "flow" |
| **Contract** | Engineering (Pact consumer) / Legal (MSA/SOW) / Sales (signed deal) / QA (contract test) | Prefix mandatory ("API contract" / "Legal contract" / "Test contract") |
| **Postmortem / retro / lesson** | SRE (blameless system) / Education (Kirkpatrick L4) / Sales (win/loss) / Agile (team retro) | Each its own word with prefix |
| **Golden dataset** | AI Engineering (labelled eval baseline) / QA (golden snapshot file) | AI = probabilistic match; QA = byte-exact. Prefix "**eval golden dataset**" / "**snapshot golden file**" |
| **Risk** | Security (exploit prob × impact) / AI Safety (catastrophic capability threshold) / Finance (runway exhaustion) / BA (impediment) | Prefix mandatory — else AI picks one and ignores rest |
| **System card vs Service blueprint vs Architecture diagram** | AI Safety / Service Design / Engineering | All three "map of how this works", but **for different audiences**. Never just "the map" |

## 7 artifacts to add to `methodology/`

1. `methodology/decisions/ADR-template.md` (Engineering — already implied by `_root/decisions.md`)
2. `methodology/runbooks/<service>.md` + `methodology/postmortems/YYYY-MM-DD-<title>.md` (SRE)
3. `methodology/security/threat-model-template.md` (Security — Shostack 4Q)
4. `methodology/ai/eval-suite-template.md` + `methodology/ai/system-card-template.md` (AI Eng + AI Safety)
5. `methodology/research/{jtbd,service-blueprint,interview-guide}-template.md` (Discovery)
6. `methodology/legal/{msa,sow,nda,dpa}.md` (Ops/Legal — pointer doc if templates in private repo)
7. `methodology/docs/diataxis-guide.md` (Education — where tutorials/how-tos/reference/explanation live)

Plus cross-school: `methodology/raci/engagement-template.md` — single RACI per client engagement, each school has owner on its artifact (Engineering Lead → SLO, Legal → MSA, AI Eng → eval suite, Brand → voice doc).

## Onboarding cheat sheets (lazy)

13 role-specific one-pagers in `methodology/onboarding/`, each ≤30 minute read, answering 5 questions: (1) what I write, (2) what I don't, (3) 7 terms for first week, (4) where to see live example, (5) who/what to ask.

Lazy creation — write cheat sheet when role actually appears in network. Pre-writing all 13 = premature optimization.

---

## Sources

### Engineering / SRE
- [Google SRE Book — Embracing Risk](https://sre.google/sre-book/embracing-risk/)
- [Google SRE Workbook — Error Budget Policy](https://sre.google/workbook/error-budget-policy/)
- [Team Topologies — Key Concepts](https://teamtopologies.com/key-concepts)
- [Staff Engineer (Will Larson) — Archetypes](https://staffeng.com/guides/staff-archetypes/)
- [DORA — Software Delivery Performance Metrics](https://dora.dev/guides/dora-metrics/)

### Security & Risk
- [Adam Shostack — Four Question Frame](https://github.com/adamshostack/4QuestionFrame)
- [Shostack — Ultimate Beginner's Guide to Threat Modeling](https://shostack.org/resources/threat-modeling)
- [OWASP — Threat Modeling Process](https://owasp.org/www-community/Threat_Modeling_Process)
- [MITRE ATT&CK](https://attack.mitre.org/)
- [WeAreBrain — SOC2 vs ISO 27001 vs GDPR](https://wearebrain.com/blog/comparing-soc-2-iso-27001-and-gdpr-compliance/)

### Quality
- [Agile Testing (Crispin/Gregory)](https://agiletester.ca/)
- [Satisfice — Rapid Software Testing](https://www.satisfice.com/rapid-software-testing-explored)
- [Bach — Exploratory Testing 3.0](https://www.satisfice.com/blog/archives/1509)
- [Pact — Consumer-Driven Contract Testing](https://docs.pact.io/)

### Business Analysis
- [IIBA — BABOK Guide overview](https://www.iiba.org/career-resources/a-business-analysis-professionals-foundation-for-success/babok/)
- [Lean Six Sigma — Process Mapping](https://goleansixsigma.com/6-process-maps-know-choose-right-one/)

### Ops / Legal / Finance / Sales
- [Promise Legal — SaaS Agreements MSA structure 2025](https://promise.legal/startup-legal-guide/contracts/saas-agreements)
- [Ironclad — What is an MSA](https://ironcladapp.com/journal/contracts/what-is-an-msa)
- [NxCode — SaaS Financial Modeling 2026](https://www.nxcode.io/resources/news/saas-financial-modeling-101-mrr-arr-ltv-cac-explained)
- [CFO Advisors — 2025 Burn-Multiple Benchmarks](https://cfoadvisors.com/blog/2025-burn-multiple-benchmarks_-how-series-a-saas-startups-can-prove-capital-efficiency)
- [MEDDIC Methodology](https://meddicc.com/meddpicc-sales-methodology-and-process)

### Brand & Marketing
- [Marty Neumeier — The Brand Gap](https://www.martyneumeier.com/the-brand-gap)
- [April Dunford — Obviously Awesome](https://www.aprildunford.com/books)
- [A List Apart — The Discipline of Content Strategy](https://alistapart.com/article/thedisciplineofcontentstrategy/)

### Research & Discovery
- [Erika Hall — Just Enough Research](https://www.uxmatters.com/mt/archives/2019/02/sample-chapter-just-enough-research.php)
- [User Interviews — JTBD field guide](https://www.userinterviews.com/ux-research-field-guide-chapter/jobs-to-be-done-jtbd-framework)
- [This Is Service Design Doing (Stickdorn)](https://www.thisisservicedesigndoing.com/)
- [Wikipedia — Nudge theory](https://en.wikipedia.org/wiki/Nudge_theory)

### Education / Knowledge Transfer
- [Diataxis framework](https://diataxis.fr/)
- [Research.com — Instructional Design Models 2026 (ADDIE/Gagne/Bloom)](https://research.com/education/instructional-design-models)
- [Stack Overflow podcast — Stripe and Waymo on great documentation](https://stackoverflow.blog/2022/06/21/an-engineers-field-guide-to-great-technical-writing-ep-455/)

### AI Engineering
- [Hamel Husain — Your AI Product Needs Evals](https://hamel.dev/blog/posts/evals/)
- [Eugene Yan — Evaluating LLM-as-Judge](https://eugeneyan.com/writing/llm-evaluators/)
- [Maven — AI Evals for Engineers & PMs (Husain/Shankar)](https://maven.com/parlance-labs/evals)
- [Chip Huyen — AI Engineering book repo](https://github.com/chiphuyen/aie-book)

### AI Safety / Red Team
- [Anthropic — Responsible Scaling Policy](https://www.anthropic.com/responsible-scaling-policy)
- [MITRE ATLAS](https://atlas.mitre.org/)
- [NIST — AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)
- [EC-Council — EU AI Act vs NIST AI RMF vs ISO/IEC 42001](https://www.eccouncil.org/cybersecurity-exchange/responsible-ai-governance/eu-ai-act-nist-ai-rmf-and-iso-iec-42001-a-plain-english-comparison/)

### SDD bridge
- [OpenSpec — What is Spec-Driven Development](https://openspec.pro/spec-driven-development/)
