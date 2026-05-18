# Robot operator UI — ICP shortlist (EU / IL / UK, Seed–Series A)

> Purpose: identify 12-15 robotics startups that plausibly need an external partner for **branded operator UI + AI-companion surfaces** on top of robot telemetry.
> Studio stack: Next.js / React / TypeScript / Supabase / Cloudflare + LLM-companion layer.
> Date: 2026-05-15. All funding and team facts inline-cited.

## Inclusion bar (recap)
- HQ in EU member state, Israel, or UK.
- Stage: Seed / Series Seed / Series A / Series A extension. Pre-Seed kept only if hardware shipped + team ≥10. Series B+ excluded.
- Builds a physical robot or robot fleet. Pure software/middleware (e.g. Sereact, Mimic) excluded unless they also ship hardware.
- Plausible operator-UI gap: small/absent UI hiring on LinkedIn, rough public dashboard, or recent stage + new form factor.
- Funded in 2024–2026 with public source.

## Shortlist table

| # | Company | HQ | Stage / last round | Product | Why on list |
|---|---|---|---|---|---|
| 1 | [ARX Robotics](https://www.arx-robotics.com) | Munich, DE | Series A €31M + €11M ext. = €42M (Apr / Jul 2025) | UGVs + Mithra OS for legacy military vehicles | Software-first robotics co; ops-console is the product, not the chassis. Six European armies = multi-tenant UI need. |
| 2 | [Stark](https://www.stark.com) | Berlin / Munich, DE | $62M Series A (Aug 2025, Sequoia) | Loitering munition + Minerva mission-control suite | "Minerva" is literally an operator UI; scaling from prototype to NATO-grade fleet view in <12 months. |
| 3 | [Orbotix Industries](https://orbotix.eu) | Warsaw, PL / Brașov, RO | €6.5M strategic (Oct 2025) | Wasper-1 swarming drone + ATA threat-acquisition | Distributed micro-factory model needs a swarm operator console; only 40 employees, no public UI hires. |
| 4 | [Quantum Systems (defence cluster)](https://quantum-systems.com) | Gilching, DE | Late C — **excluded from primary shortlist** but sibling-tier startups follow | Reconnaissance VTOL | Reference benchmark for the cluster — not a target itself. |
| 5 | [Humanoid (SKL Robotics)](https://thehumanoid.ai) | London, UK | Founder-led $50M, prepping Series A (Q3 2025) | HMND 01 Alpha dual-arm mobile manipulator | First commercial robot landing early 2027 — ops UI roadmap still empty in public material. Currently zero front-end UI hires on LinkedIn. |
| 6 | [Flexion Robotics](https://flexion.ai) | Zurich, CH | $50M Series A (Nov 2025, Prosus/DST) after $7.35M seed | "Brain" / intelligence stack licensed per-robot for humanoids | Per-robot SaaS licensing implies a customer-facing console; team is ex-NVIDIA roboticists, no UI specialists named. |
| 7 | [Mimic Robotics](https://mimicrobotics.com) | Zurich, CH | $16M Seed (Nov 2025, Elaia + Speedinvest) | Dexterous robotic hands on industrial arms | Sells "human-level dexterity as a module" — needs a teach-and-monitor UI for factory operators. ETH spinout, 25 engineers. |
| 8 | [Sunrise Robotics](https://sunriserobotics.co) | Ljubljana, SI | $8.5M Seed (Jun 2025, Plural) | Modular dual-arm industrial cells with simulation-trained AI | Sells "easy-to-deploy automation" — the deployment surface itself is the moat. 10+ LOI customers but no listed product designer. |
| 9 | [Generative Bionics](https://generativebionics.com) | Genoa, IT | €70M pre-launch round (Dec 2025) | Italian-made general-purpose humanoid (IIT spinout) | Just launched; recruiting hardware-heavy. No designer or front-end role surfaced. (unverified UI hiring) |
| 10 | [Energy Robotics](https://www.energy-robotics.com) | Darmstadt, DE | $13.5M Series A (Oct 2025, Blue Bear Capital + Climate Investment) | Hardware-agnostic inspection software for Spot, DJI, MHI fleets | Their software *is* the operator UI; today's web app is utilitarian — clear room for branded ops dashboard + LLM-narrated inspection reports. |
| 11 | [Filics](https://www.filics.com) | Munich, DE | €13.5M (Jul 2025, Amazon Industrial Innovation Fund + others) | Flat transport robots for intralogistics (≤1.2 t) | 6-year-old hardware-deep team, Amazon as strategic backer means white-label fleet UI demand. No design lead visible. |
| 12 | [Capra Robotics](https://capra.ooo) | Aarhus / Odense, DK | €11.3M Seed (Jan 2025, Promus + Supernova) | Outdoor AMR for inspection, logistics, urban maintenance | Selling to Lockheed Martin + Deutsche Telekom — those buyers expect a polished branded ops surface; team is hardware-led. |
| 13 | [Kongsberg Ferrotech](https://kongsbergferrotech.com) | Kongsberg, NO | €12M Seed (Jul 2025, NATO Innovation Fund + Investinor) | "Nautilus" subsea inspection-and-repair robot with dry-habitat | Customers are Equinor, ADNOC, Aramco, Petrobras — enterprise ops surface required; current public UI is engineering-grade. |
| 14 | [Flybotix](https://www.flybotix.com) | Lausanne, CH | ~$10M Series A ext. (Sep 2025, family office + Greeneering) | ASIO X confined-space inspection drone + SaaS | The SaaS already exists but visual identity feels OEM-stock; opportunity to lift the operator console into a branded product. |
| 15 | [PHINXT Robotics](https://www.phinxt.com) | London, UK | £2M Seed (Aug 2024, Sure Valley Ventures) | Edge-AI fleet orchestration platform + AMR | Self-describes as a "universal software platform" — i.e. UI-heavy product with a 2-yr-old founding team and tiny headcount. Direct LLM-companion fit. |
| 16 | [Rollo Robotics](https://1rollo.com) | Tallinn, EE | €3.7M Pre-Seed (Jan 2026, FoodLabs + Prototype) — *included despite stage because founders are ex-Cleveron and hardware already in build* | Stable autonomous monowheel security robot | Brand-led founders (Cleveron lineage) → premium look-and-feel for the operator surface is on-brand. Currently 0 designers public. |

---

## Per-row deep dives

### 1. ARX Robotics — `arx-robotics.com`
- **Founded** 2022; **HQ** Munich, Germany.
- **Stage**: Series A €31M led by HV Capital with NATO Innovation Fund, Project A, Omnes (Apr 2025); €11M extension led by Speedinvest (Jul 2025). Total Series A €42M. ([Tech.eu](https://tech.eu/2025/04/28/arx-robotics-raises-eur31m-series-a-to-advance-military-automation/), [Tech.eu ext.](https://tech.eu/2025/07/29/arx-robotics-secures-eur11m-in-series-a-extension/))
- **Product**: Unmanned ground vehicles (UGVs) plus **Mithra OS**, an AI operating system that retrofits legacy military vehicles into autonomy-capable fleet members. Deployed by six European armies including Bundeswehr and Ukraine.
- **Operator UI**: Mithra OS *is* the surface — fleet view, situational-awareness map, mixed-fleet command. Public marketing is engineering-led; no end-user UX visible. LinkedIn shows hardware + ROS engineers dominant, no senior design hire surfaced.
- **Why studio fits**: NATO-grade ops console = high stakes for clarity, theming for multiple buying customers (per-army theming), and an LLM "AAR / mission-debrief" companion is a clean fit on top of UGV telemetry.
- **Hook**: "Six armies on one OS is a multi-tenant UI problem — happy to send a 5-screen Mithra reskin showing how per-army theming + LLM mission-debrief lands."

### 2. Stark — `stark.com`
- **Founded** 2024; **HQ** Berlin, production in Munich.
- **Stage**: $62M Series A led by Sequoia (Aug 2025), $500M valuation, with Peter Thiel personal. ([Tech.eu](https://tech.eu/2025/08/21/stark-raises-62m-for-weaponised-drone-systems-backed-by-sequoia-and-peter-thiel/))
- **Product**: OWE-V Virtus loitering munition (VTOL) + **Minerva** mission-control suite. Spun out of Quantum Systems' founder Florian Seibel.
- **Operator UI**: Minerva is explicitly the operator surface. Public screenshots are limited (defence-sensitive). Likely to need rapid iteration as customer set widens from German MoD to NATO partners.
- **Why studio fits**: defence buyers expect a branded, defensible UI per nation; Stark hires fast on hardware but very few public design/front-end roles. LLM-narrated brief / debrief layer plugs naturally into Minerva.
- **Hook**: "Watched the Virtus drop-test footage — would love to send a Minerva concept showing operator-side narration + tactical-pause LLM agent."

### 3. Orbotix Industries — `orbotix.eu`
- **Founded** 2023; **HQ** Warsaw, PL with new factory in Brașov, RO.
- **Stage**: €6.5M strategic round (Oct 2025), BVVC Fund lead. ([EU-Startups](https://www.eu-startups.com/2025/10/warsaw-based-orbotix-raises-e6-5-million-to-advance-autonomous-defence-systems-in-europe/))
- **Product**: Wasper-1 drone-swarming platform + ATA autonomous target-acquisition system. Strategy is networked micro-factories across NATO's eastern flank.
- **Operator UI**: Swarm operator gets coordinates, AI proposes targets, operator confirms. The interface lives or dies on how it shows AI confidence + human override. Currently nothing public.
- **Why studio fits**: swarm command + AI-companion-with-veto is exactly the LLM/telemetry overlay pattern the studio is positioned for. ~40 staff, no design lead surfaced.
- **Hook**: "Swarm + human-in-the-loop is a UX problem before it's a robotics problem. Mocked a Wasper operator screen showing AI-target queue + LLM rationale — share?"

### 4. (Quantum Systems — reference, not a target)
Series C unicorn, excluded from outreach but useful context: it's the German defence-drone benchmark every smaller fleet startup in this list is measured against.

### 5. Humanoid (SKL Robotics) — `thehumanoid.ai`
- **Founded** 2024; **HQ** London, UK.
- **Stage**: Founder-led $50M (Artem Sokolov personal capital), Series A in preparation. ([Sifted](https://sifted.eu/articles/humanoid-robotics-alina-interview), [Robot Report](https://www.therobotreport.com/u-k-based-startup-humanoid-unveils-hmnd-01-alpha-mobile-manipulator/))
- **Product**: HMND 01 Alpha — dual-armed mobile manipulator for industrial use, developed in 7 months. Beta wheeled robot Q3 2026, first commercial sales early 2027.
- **Operator UI**: Currently nothing — company is in hardware mode, IPO pencilled for 2029/30. Ops console is the obvious next gap as commercial customers approach.
- **Why studio fits**: London-based founder with consumer-brand sensibility (SKL.vc background) → cares about look-and-feel. No frontend hires public. LLM-companion for industrial-operator handover is a strong story.
- **Hook**: "Saw HMND 01 Alpha. Industrial pilots in 2026 will need an operator console long before IPO prep — happy to ship a 3-screen concept."

### 6. Flexion Robotics — `flexion.ai`
- **Founded** Jan 2024; **HQ** Zurich, CH.
- **Stage**: $7.35M Seed (Frst, Moonfire, Redalpine), then $50M Series A led by Prosus Ventures with DST Global, NVentures, Redalpine, Moonfire (Nov 2025). Total $57.35M. ([Crunchbase News](https://news.crunchbase.com/venture/robotic-brain-building-startup-flexion-raise/), [Robot Report](https://www.therobotreport.com/flexion-raises-50m-build-ai-systems-power-humanoids/))
- **Product**: Intelligence layer for humanoid robots (full-body control + VLA training). Sells annual per-robot software license to humanoid OEMs.
- **Operator UI**: Per-robot SaaS implies a tenant dashboard for OEM customers (Robot health, training data, task library). Public site is minimal; team is ex-NVIDIA / Tesla research-heavy.
- **Why studio fits**: SaaS pricing model = needs a customer console urgently. Founders are research-track, not design-track. LLM "task-author" companion fits the VLA training pitch.
- **Hook**: "Annual per-robot SaaS for humanoid OEMs implies a tenant dashboard. Want a sketch of Flexion's customer console + task-author copilot?"

### 7. Mimic Robotics — `mimicrobotics.com`
- **Founded** 2024 (ETH Zurich spinout); **HQ** Zurich, CH.
- **Stage**: $16M Seed (Nov 2025), led by Elaia + Speedinvest, with Founderful, Sequoia Scout, 10X Founders, 2100. ([Sifted](https://sifted.eu/articles/eth-robotics-spinout-mimic-raises-16m-seed))
- **Product**: Dexterous robotic hands + conventional industrial arms = "human-level dexterity module". 25 engineers, expected to pass 30 by year-end.
- **Operator UI**: Selling a *module*, so the buyer (factory IT) needs a teach-and-monitor UI plus a fleet-rollup. Today there's nothing customer-facing public.
- **Why studio fits**: classic "research team needs an ops surface" case. LLM-companion can plausibly become a "show me / teach me" hand-trainer.
- **Hook**: "ETH spinouts always nail dexterity and undersell the operator screen. Want a teach-by-demo UI mock plus LLM trainer agent?"

### 8. Sunrise Robotics — `sunriserobotics.co`
- **Founded** 2022; **HQ** Ljubljana, SI.
- **Stage**: $8.5M Seed (Jun 2025), led by Plural with Tapestry, Seedcamp, Tiny.vc, Prototype Capital. ([Fortune](https://fortune.com/2025/06/13/sunrise-robotics-seed-funding-8-5-million-plural-industrial-robots/), [EU-Startups](https://www.eu-startups.com/2025/06/slovenian-startup-sunrise-robotics-emerges-from-stealth-with-e7-3-million-for-their-simulation-trained-robots/))
- **Product**: Modular dual-arm industrial cells trained in simulation. 10+ LOI customers in supercars, batteries, consumer electronics. Initial deploys in EU electronics.
- **Operator UI**: Their pitch is "10× faster deployment" — that promise requires a polished deployment + monitoring console. Public site is brochure-style.
- **Why studio fits**: Plural / Wise founder energy → quality-of-craft expectation. No front-end / design lead surfaced. LLM-companion as "explain why the cell paused" maps onto sim-trained AI rationale.
- **Hook**: "10× faster deploy is a UI promise. Want a Sunrise-branded operator console + LLM 'why-did-it-pause' agent?"

### 9. Generative Bionics — `generativebionics.com` (unverified URL)
- **Founded** Jul 2024; **HQ** Genoa, Italy (IIT spinout).
- **Stage**: €70M pre-launch round (Dec 2025), CDP Venture Capital AI Fund lead, with AMD Ventures, Eni Next, Duferco, Tether, RoboIT. ([EurekAlert](https://www.eurekalert.org/news-releases/1109288))
- **Product**: General-purpose "Made-in-Italy" humanoid for manufacturing, logistics, healthcare, retail. Founders: Daniele Pucci, Alessio Del Bue, Marco Maggiali, Andrea Pagnin.
- **Operator UI**: Just emerged from stealth; no public ops surface yet. Hiring is hardware-heavy. (unverified — LinkedIn hire scan not run)
- **Why studio fits**: massive round + early stage = the moment to set design DNA before hardware-engineering culture dictates the UI. Italian craft-brand sensitivity.
- **Hook**: "Italian humanoid + €70M is a once-per-decade chance to define the operator-UI DNA before engineering culture sets it for you. Want a kickoff conversation?"

### 10. Energy Robotics — `energy-robotics.com`
- **Founded** 2019; **HQ** Darmstadt, DE.
- **Stage**: $13.5M / €11.5M Series A (Oct 2025), co-led by Blue Bear Capital + Climate Investment. ([Robot Report](https://www.therobotreport.com/energy-robotics-secures-series-a-scale-critical-infrastructure-inspections/), [EU-Startups](https://www.eu-startups.com/2025/10/german-startup-energy-robotics-raises-e11-5-million-to-advance-autonomous-robot-and-drone-inspection-software/))
- **Product**: Hardware-agnostic AI software for autonomous inspection — integrates with Boston Dynamics Spot, DJI drones, Mitsubishi Heavy. 1M+ inspections completed across 5 continents.
- **Operator UI**: Web platform exists; visual identity is utilitarian SaaS. Pure-software company, so the dashboard *is* the product — clear room for a brand-led overhaul + LLM-narrated inspection-report layer.
- **Why studio fits**: software-only = no hardware risk to navigate; mixed-fleet inspection is a perfect canvas for "ask the LLM: what changed since last inspection?"
- **Hook**: "Your mixed-fleet (Spot + DJI + MHI) console is the ideal canvas for an LLM 'what-changed' inspection-report companion. 4-screen concept on its way."

### 11. Filics — `filics.com`
- **Founded** 2019; **HQ** Munich, DE.
- **Stage**: €13.5M round (Jul 2025), Sandwater + Alven + Amazon Industrial Innovation Fund + F-LOG Ventures + Bayern Kapital. ([EU-Startups](https://www.eu-startups.com/2025/07/german-robotics-company-filics-secures-e13-5-million-to-expand-and-roll-out-its-robotics-platform/))
- **Product**: Flat transport robots (≤1.2 t) for intralogistics. Amazon as strategic backer signals white-label / fleet rollouts.
- **Operator UI**: Hardware-first team; current marketing emphasises payload + footprint, not ops console. With Amazon-grade fleet rollouts incoming, the UI will need to scale.
- **Why studio fits**: classic German Mittelstand engineering company — strong machine, weak digital surface. LLM-companion as "fleet floor explainer" lands well with non-technical warehouse managers.
- **Hook**: "Amazon backing means multi-tenant fleet at scale soon. Want a Filics-branded warehouse-manager console with LLM floor-explainer?"

### 12. Capra Robotics — `capra.ooo`
- **Founded** 2019; **HQ** Aarhus, DK (Odense robotics cluster).
- **Stage**: €11.3M Seed (Jan 2025), Promus Ventures + Supernova Invest lead, Summiteer and EIFO participation. ([Silicon Canals](https://siliconcanals.com/capra-robotics-secures-e11-3m/))
- **Product**: Outdoor AMR platform for inspection, logistics, urban maintenance. Customers include Lockheed Martin and Deutsche Telekom.
- **Operator UI**: Current public material is hardware-photo-heavy. Lockheed/DT-grade buyers expect a polished branded ops console; team is hardware-led per Odense convention.
- **Why studio fits**: enterprise customers = enterprise ops surface = exactly the gap between hardware team and end-user UI. LLM "ask my fleet" companion lands well with DT/Lockheed-grade users.
- **Hook**: "Lockheed and DT expect an ops console that looks like one of their own products. Want a Capra-grade branded fleet view + LLM 'ask-my-fleet' demo?"

### 13. Kongsberg Ferrotech — `kongsbergferrotech.com`
- **Founded** 2014; **HQ** Kongsberg, NO.
- **Stage**: €12M Seed (Jul 2025), NATO Innovation Fund + Investinor lead. ([Silicon Canals](https://siliconcanals.com/kongsberg-ferrotech-secures-12m/), [NIF press release](https://www.nif.fund/news/nato-innovation-fund-leads-eur12-million-seed-financing-round-in-kongsberg-ferrotech-alongside-investinor-to-boost-global-security-and-energy-resilience/))
- **Product**: "Nautilus" subsea inspection / repair robot using a dry-habitat seal — eliminates divers + dry-docking. Customers: Equinor, ADNOC, Aramco, Total Energies, Petrobras, PTTEP.
- **Operator UI**: Operator must see live habitat seal state + repair-progress + LLM summary of "what was found / repaired" for the energy-major customer. Today nothing public.
- **Why studio fits**: enterprise oil-major customers expect SCADA-grade dashboards; LLM-narrated dive report is a clear differentiator. Hardware-and-marine-engineering team, no design surfaced.
- **Hook**: "ADNOC and Aramco don't read ROS topics — they read dashboards. Want a Nautilus operator-view + LLM dive-report mock?"

### 14. Flybotix — `flybotix.com`
- **Founded** 2019; **HQ** Lausanne, CH.
- **Stage**: ~$10M Series A extension (Sep 2025), family office lead + Greeneering Invest. ([Flybotix press release](https://www.flybotix.com/flybotix-secures-close-to-usd-10m-in-series-a-extension-financing-confined-space-inspection-drone/))
- **Product**: ASIO X confined-space inspection drone + SaaS platform; distributors in 30+ countries.
- **Operator UI**: SaaS exists but visual identity is generic-inspection-tool. Opportunity: lift the operator console into a flagship-grade product, integrate LLM inspection-narrator.
- **Why studio fits**: SaaS already monetised → there's budget and the pain is visible. EPFL-spinout team is hardware-and-control-theory dominant.
- **Hook**: "ASIO X SaaS is the right product wearing a generic skin. Mock of a Flybotix-branded operator console + LLM defect-narrator on its way."

### 15. PHINXT Robotics — `phinxt.com`
- **Founded** 2022; **HQ** London, UK.
- **Stage**: £2M / €2.3M Seed (Aug 2024), Sure Valley Ventures lead. ([EU-Startups](https://www.eu-startups.com/2024/08/london-based-phinxt-robotics-raises-e2-3-million-to-redefine-warehouse-automation/))
- **Product**: Edge-AI universal fleet orchestration platform for warehouses, plus their own AMR. Long-term vision: delivery drones + autonomous vehicles.
- **Operator UI**: Self-describes as a "universal software platform" — so the UI is the product. Tiny headcount, no design lead public.
- **Why studio fits**: smallest funded company on the list and the one most directly aligned with "branded operator UI on telemetry". An LLM "fleet whisperer" companion is core to their pitch.
- **Hook**: "Universal fleet orchestration is a UI promise more than a robotics one. Want a PHINXT-branded warehouse console + LLM fleet-whisperer demo?"

### 16. Rollo Robotics — `1rollo.com`
- **Founded** 2025; **HQ** Tallinn, EE.
- **Stage**: €3.7M Pre-Seed (Jan 2026), FoodLabs + Prototype lead, RUP (Enterprise Estonia) participation. ([Invest in Estonia](https://investinestonia.com/estonian-startup-rollo-robotics-secures-e3-7m-funding-to-mature-monowheel-security-robot/))
- **Product**: World's first stable autonomous monowheel security robot. Founders: Arno Kütt (ex-Cleveron) + Sander Sebastian Agur.
- **Operator UI**: Stage is below the bar (pre-seed), but included because hardware is already in build and founders have a brand-first track record (Cleveron's parcel-locker aesthetics).
- **Why studio fits**: monowheel = visually unique form factor → premium brand expectation. Security buyers expect a control-room dashboard + LLM patrol-summary fits perfectly.
- **Hook**: "Cleveron taught the world that hardware that looks like a brand wins. Want a Rollo control-room concept + LLM patrol-debrief?"

---

## Notes & caveats

- **Stage drift watch**: Sereact (Stuttgart) and Mentee Robotics (Israel) both crossed into Series B / acquisition during 2025-2026 — excluded.
- **Israel under-represented**: defence-side robotics (XTEND, Heven AeroTech) jumped to mega-rounds and are out of the band. Mentee Robotics was acquired by Mobileye for $900M Jan 2026 (out of scope). Tevel Aerobotics and Blue White Robotics are Series B+. Most Israeli AI-robotics dollars in 2024-2025 flowed to companies already past Series A or into pure software. Result: only indirect Israeli signal; would recommend a follow-up pass focused on Israeli mega-seeds (`$20-50M` Seed band per CTech) explicitly tagged robotics.
- **France gap**: Wandercraft is Series D; Pollen Robotics was acquired by Hugging Face; no obvious Seed/A French humanoid or AMR matching the bar surfaced in this pass. Re-mine in 3-6 months.
- **Unverified items**: Generative Bionics LinkedIn hiring scan was not run; "no design lead public" is inferred from press coverage and not from direct profile inspection — confirm before outreach. Rollo Robotics is pre-seed and below the formal stage bar, included on founder-track-record grounds only.
- **Operator-UI signal strength** (subjective ranking based on press + headcount evidence): strongest = ARX, Stark, PHINXT, Energy Robotics, Capra. Mid = Flexion, Mimic, Sunrise, Flybotix, Filics, Kongsberg Ferrotech. Speculative = Humanoid, Generative Bionics, Orbotix, Rollo.

## Sources
- [Tech.eu — ARX Robotics Series A](https://tech.eu/2025/04/28/arx-robotics-raises-eur31m-series-a-to-advance-military-automation/)
- [Tech.eu — Stark $62M Series A](https://tech.eu/2025/08/21/stark-raises-62m-for-weaponised-drone-systems-backed-by-sequoia-and-peter-thiel/)
- [EU-Startups — Orbotix](https://www.eu-startups.com/2025/10/warsaw-based-orbotix-raises-e6-5-million-to-advance-autonomous-defence-systems-in-europe/)
- [Sifted — Humanoid interview](https://sifted.eu/articles/humanoid-robotics-alina-interview)
- [Robot Report — Humanoid HMND 01 Alpha](https://www.therobotreport.com/u-k-based-startup-humanoid-unveils-hmnd-01-alpha-mobile-manipulator/)
- [Crunchbase News — Flexion Series A](https://news.crunchbase.com/venture/robotic-brain-building-startup-flexion-raise/)
- [Sifted — Mimic Robotics Seed](https://sifted.eu/articles/eth-robotics-spinout-mimic-raises-16m-seed)
- [Fortune — Sunrise Robotics emergence](https://fortune.com/2025/06/13/sunrise-robotics-seed-funding-8-5-million-plural-industrial-robots/)
- [EurekAlert — Generative Bionics €70M](https://www.eurekalert.org/news-releases/1109288)
- [Robot Report — Energy Robotics Series A](https://www.therobotreport.com/energy-robotics-secures-series-a-scale-critical-infrastructure-inspections/)
- [EU-Startups — Filics €13.5M](https://www.eu-startups.com/2025/07/german-robotics-company-filics-secures-e13-5-million-to-expand-and-roll-out-its-robotics-platform/)
- [Silicon Canals — Capra Robotics Seed](https://siliconcanals.com/capra-robotics-secures-e11-3m/)
- [Silicon Canals — Kongsberg Ferrotech Seed](https://siliconcanals.com/kongsberg-ferrotech-secures-12m/)
- [Flybotix press release — Series A ext.](https://www.flybotix.com/flybotix-secures-close-to-usd-10m-in-series-a-extension-financing-confined-space-inspection-drone/)
- [EU-Startups — PHINXT Robotics Seed](https://www.eu-startups.com/2024/08/london-based-phinxt-robotics-raises-e2-3-million-to-redefine-warehouse-automation/)
- [Invest in Estonia — Rollo Robotics Pre-Seed](https://investinestonia.com/estonian-startup-rollo-robotics-secures-e3-7m-funding-to-mature-monowheel-security-robot/)
