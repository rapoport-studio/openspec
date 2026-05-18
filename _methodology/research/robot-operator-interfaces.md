# Robot Operator Interfaces — Vertical Scan for Rapoport Studio

> Reference document. Compiled May 2026. Scope: branded operator UI / fleet dashboards / AI-companion surfaces for robot makers (humanoid, AMR, drone, cobot, quadruped, service, last-mile). Stack assumption: Next.js 16 / React 19 / TypeScript / Tailwind / Supabase / Cloudflare Workers, plus an LLM-companion layer ("Muse Architect" style).

## 1. Robot inventory — 7 classes × top 3 vendors

The inventory below covers ~21 platforms across 7 robot classes. For each platform the "Command API" column lists the public surface a Gateway layer would actually integrate against; the "SDK public?" column reflects what an outside developer can do without an OEM partnership; the "Operator UI today" column captures whether the OEM already ships an operator UI or relies on third parties (Foxglove, Formant, InOrbit) or hand-rolled white-label.

### 1.1 AMR — Autonomous Mobile Robots (logistics, intralogistics)

| Vendor + product | Class | Command API | SDK public? | Operator UI today | Notable gap |
|---|---|---|---|---|---|
| **[Locus Robotics — LocusONE / Locus Origin / Vector](https://locusrobotics.com/locusone)** | AMR (piece-pick, case-pick) | LocusServer cloud orchestrator + REST integrations to WMS/ERP; VDA 5050 supported on certain SKUs ([source](https://logisticsbusiness.com/materials-handling/amr-agv/locus-robotics-attracts-dollar117m-funding-2/)) | Partner-gated; LocusONE is a closed SaaS surface | LocusOne SaaS dashboard (in-house). 230+ sites, up to 500 bots/site ([source](https://research.contrary.com/company/locus-robotics)) | "Fleet" view is operations-grade, not employee-facing. No companion narrative for warehouse leads; no AI post-mortem of pick anomalies. |
| **[Fetch Robotics / Zebra Technologies — FetchCore](https://www.zebra.com/us/en/about-zebra/newsroom/press-releases/2021/zebra-technologies-to-acquire-fetch-robotics.html)** | AMR (warehouse) | FetchCore cloud APIs + WMS/ERP connectors; ROS 2 under the hood | Partner-gated. Zebra is **winding down** the Fetch AMR group ([source](https://www.therobotreport.com/zebra-technologies-winding-down-fetch-based-mobile-robot-group/)) | FetchCore web console (legacy) | Customers about to lose support — open displacement window for white-label replacement UIs on existing Fetch hardware. |
| **[Mobile Industrial Robots (MiR) — MiR Fleet Enterprise](https://mobile-industrial-robots.com/products/software/mir-fleet)** | AMR (logistics) | REST APIs (pub/sub model since 2024); WMS/ERP/MES connectors; VDA 5050 support announced ([source](https://control.com/news/mir-launches-scalable-cybersecurity-enhanced-autonomous-mobile-robot-amr-fleet-management-software/)) | Public SDK + REST docs available to MiR Distribution partners | MiR Fleet (in-house, web). Supports up to 100 robots, SSO, role-based access | Fleet UI is utilitarian; mixed-vendor floors (MiR + UR cobot + something else) need an aggregating dashboard MiR doesn't ship. |
| **[OTTO Motors / Rockwell Automation — OTTO Fleet Manager](https://ottomotors.com/)** | AMR (heavy industrial) | Pre-built REST APIs to MES/ERP/WMS ([source](https://ottomotors.com/index.html)); proprietary protocol on robot | Partner-gated under Rockwell channel | OTTO Fleet Manager (in-house web) | Rockwell-acquired in Oct 2023; UI is being re-platformed. Brittle window for customer-branded dashboards over the next 18-24 months. |
| **[Geek+ (Geekplus) — Robot Management System (RMS)](https://www.geekplus.com/en)** | AMR (goods-to-person, shelf-to-person) | API integration layer for external systems; proprietary cloud orchestrator | Partner-gated; Geekplus listed on HK Exchange 2025 | RMS (in-house, web); 23% global order-fulfilment AMR market share, 48.5% in shelf-to-person ([source](https://www.geekplus.com/resources/news/global-warehouse-automation-surges-as-geekplus-extends-no.1-amr-leadership-for-seven-consecutive-years)) | Mostly Chinese-market UX patterns; EU/IL/MD clients want localized, branded surfaces with VDA 5050 interop. |

### 1.2 Humanoid — Bipedal general-purpose

| Vendor + product | Class | Command API | SDK public? | Operator UI today | Notable gap |
|---|---|---|---|---|---|
| **[Figure AI — Figure 02](https://www.figure.ai/)** | Humanoid (logistics, manufacturing) | Vision-Language-Action model controls upper body at 200 Hz; "open SDKs and APIs" for WMS integrations ([source](https://humanoidspecs.com/robots/figure-02)) | Open SDK announced but partner-controlled in practice; BMW pilot is the showcase ([source](https://www.prnewswire.com/news-releases/figure-unveils-figure-02-its-second-generation-humanoid-setting-new-standards-in-ai-and-robotics-302214889.html)) | In-house Helix model + supervisor UI; no public branded surface | No "trust calibration" UX — operator can't see why the VLA model picked an action. AI-companion explainer layer is wide open. |
| **[Apptronik — Apollo](https://apptronik.com/apollo)** | Humanoid (logistics, warehouse) | API + SDK for programming; "point-and-click control" software suite; modular (legs/wheels/fixed base) ([source](https://humanoid.guide/product/apollo/)) | SDK available to design partners (Mercedes-Benz, GXO, NASA Apollo Accord) | "Apollo software suite" — operator console for task setup, telemetry. In-house. | Multi-Apollo operator UX is still nascent. Companion layer that narrates "what Apollo just did and why" is uncovered. |
| **[1X Technologies — Neo (consumer) / Neo Beta (industrial)](https://www.1x.tech/)** | Humanoid (home + light commercial) | NEO Cortex (NVIDIA Jetson Thor); Redwood embodied AI model; WiFi/BT/5G for remote teleop ([source](https://parametric-architecture.com/neo-by-1x-worlds-first-humanoid-robot/)) | No public SDK at launch (Oct 2025); $20k Early Access or $499/mo ([source](https://humanoidspecs.com/robots/1x-neo)) | 1X-controlled teleoperator pool ("Expert Mode"); branded "1X" surface | Consumer-grade transparency UX (what is the operator seeing? what's recorded?) is missing — privacy-first companion layer is a fit. |
| **[Tesla — Optimus](https://www.tesla.com/AI)** | Humanoid (Tesla-internal) | Not externally accessible. FSD-stack inheritance. V3 expected late July/Aug 2026 ([source](https://humanoid.guide/product/optimus/)) | None public; internal Tesla R&D only | None public | N/A for studio — Tesla is closed. Listed for completeness. |
| **[Unitree — G1 / H1](https://www.unitree.com/g1)** | Humanoid (research, low-cost commercial) | `unitree_sdk2` (C++/Python); CycloneDDS middleware; high-level `LocoClient` JSON-RPC; low-level motor control via `rt/arm_sdk` ([source](https://support.unitree.com/home/en/G1_developer)) | Fully open SDK on [GitHub](https://github.com/unitreerobotics) | None — Unitree expects users to build their own UI | **Largest open opportunity in the inventory.** Cheap Unitree fleets in academia + early-commercial pilots have no production UI. Branded operator surface + AI companion = clear value. |

### 1.3 Cobot — Collaborative robot arms

| Vendor + product | Class | Command API | SDK public? | Operator UI today | Notable gap |
|---|---|---|---|---|---|
| **[Universal Robots — UR3e / UR5e / UR10e / UR16e / UR20 + PolyScope X](https://www.universal-robots.com/)** | Cobot (industrial arm, payloads 3-20 kg) | URScript (proprietary script); URCap SDK (Java/Swing); RESTful Robot API on PolyScope X — start/stop programs, get/set system time, query license ([source](https://docs.universal-robots.com/PolyScopeX_SDK_Documentation/build/SDK-v0.18/BackendContributionConnectivity.html)); URCap REST exposed via ingress on port 80 with `/<vendorid>/<urcapId>/<endpoint>` ([source](https://docs.universal-robots.com/PolyScopeX_SDK_Documentation/build/SDK-v0.18/BackendContributionConnectivity.html)) | Fully public; UR+ partner ecosystem | PolyScope X (in-house teach pendant + tablet UI) | Operator UI is per-arm; multi-arm orchestration across a UR fleet (10+ cells) is unsolved — customers build it themselves. |
| **[Franka Robotics — Franka Research 3 / Production 3](https://franka.de/franka-research-3)** | Cobot (research-grade, 7-DOF) | `libfranka` (C++) + `pylibfranka` (Python ≥0.16.0); Franka Control Interface (FCI) at 1 kHz position/velocity/torque ([source](https://github.com/frankaemika)); ROS 2 Jazzy support in `franka_ros2` v3.0.0+ | Fully public + ROS 2 first-class | Desk (in-house web UI); academic-leaning | Production-3 customers need an industrial branded surface — Desk is research-flavored. AI-assisted skill chaining UX is uncovered. |
| **[KUKA — LBR iiwa / iisy (cobot range)](https://www.kuka.com/en-us/products/robotics-systems/software/system-software/kuka_systemsoftware)** | Cobot (industrial arm) | KUKA Robot Language (KRL, Pascal-like) ([source](https://en.wikipedia.org/wiki/KUKA_Robot_Language)); KUKA System Software (KSS) operating system; iiQKA OS for newer hardware with Creator Developer Tools | Partner-gated; iiQWorks free tiers exist ([source](https://www.kuka.com/en-us/future-production/iiqka-robots-for-the-people/creator-portal/creator-developer-tools)) | smartPAD (teach pendant); KUKA.WorkVisual (PC) | Multi-cell / multi-vendor floors still need bespoke dashboards — KUKA's stack is single-arm-centric. |
| **[ABB — IRB cobot range + OmniCore controller](https://www.abb.com/global/en/areas/robotics/products/controllers/omnicore)** | Cobot (industrial arm) | RAPID language; Robot Web Services (RESTful, HTTP, XHTML/JSON) ([source](https://forums.robotstudio.com/discussion/12180/how-to-get-started-with-robot-web-services-and-jog-a-robot-via-rest-api-requests)); OmniCore App SDK + IoT Gateway (OPC UA); RobotStudio Cloud web programming | Partner-gated SDK; RWS is available on the controller | FlexPendant (in-house); AppStudio for partner apps; RobotStudio Cloud | OmniCore replaces IRC5 by **June 2026** ([source](https://new.abb.com/news/detail/116279/abb-launches-next-generation-robotics-control-platform-omnicore)) — large installed base must migrate apps; window for white-label operator surfaces during the transition. |

### 1.4 Quadruped — Four-legged inspection / patrol

| Vendor + product | Class | Command API | SDK public? | Operator UI today | Notable gap |
|---|---|---|---|---|---|
| **[Boston Dynamics — Spot](https://dev.bostondynamics.com/)** | Quadruped (inspection, public safety) | gRPC API; `RobotCommandService` (stand, sit, selfright, velocity, trajectory, safe_power_off); Python client library; payload SDK ([source](https://github.com/boston-dynamics/spot-sdk)) | Fully public Spot SDK on GitHub; ROS Spot driver maintained by community | Spot tablet controller (in-house) + Orbit (web fleet UI) | Spot Orbit is good but generic. Vertical-specific UIs (oil & gas inspection, construction progress, security patrol) are built piecemeal by SIs. |
| **[Unitree — Go2 / B2](https://www.unitree.com/go2)** | Quadruped (research, low-cost commercial) | `unitree_sdk2` (C++/Python); CycloneDDS middleware; ROS 2 bridge | Fully public SDK | None — Unitree expects users to build their own UI | Same as G1 humanoid: massive grey-market install base with no production UI. Foxglove works but is generic. |
| **[ANYbotics — ANYmal / ANYmal X (ex-proof)](https://www.anybotics.com/robotics/anymal/)** | Quadruped (industrial inspection, IP67, ATEX) | RESTful endpoints; standard file formats (JPG, WAV, E57, PLY, DAE); real-time streaming for acoustic/thermal/gas/visual; mission management; pose feedback ([source](https://www.anybotics.com/robotics/anymal/)) | Public API; partner integrations e.g. Siemens developer portal ([source](https://developer.siemens.com/anybotics/overview.html)) | ANYbotics in-house web UI; mostly tuned for oil & gas / chemical inspection | ANYmal's customers are conservative oil-and-gas operators — branded "your-refinery" dashboards beat generic ANYbotics console for executive buyers. |

### 1.5 Drone — Aerial unmanned

| Vendor + product | Class | Command API | SDK public? | Operator UI today | Notable gap |
|---|---|---|---|---|---|
| **[DJI — Matrice 4 / Matrice 400 + Dock 3](https://developer.dji.com/)** | Drone (enterprise inspection, delivery, defense-adjacent) | Cloud API (MQTT, HTTPS, WebSocket); Onboard SDK; Payload SDK; Mobile SDK; Pilot 2 + Dock connectivity ([source](https://developer.dji.com/doc/cloud-api-tutorial/en/)) | Public developer ecosystem (with regional restrictions in US/IL) | DJI FlightHub 2 (in-house cloud); third parties FlytBase, DroneDeploy, Skydio Cloud-compat ([source](https://enterprise-insights.dji.com/blog/dji-sdk-guide)) | **US/EU defense + critical-infrastructure customers actively need non-DJI-cloud, branded operator UIs** because of NDAA / EU procurement rules. Massive displacement market. |
| **[Skydio — X10 / X10D (defense)](https://www.skydio.com/)** | Drone (enterprise + defense, NDAA-compliant) | Skydio Cloud REST APIs + Webhooks; sample Python client on GitHub; Fleet Manager + Media Sync (license-gated) ([source](https://apidocs.skydio.com/reference/introduction)) | Public developer tools with sandbox ([source](https://www.skydio.com/developer-tools)) | Skydio Cloud (in-house web); Skydio Extend integrations (DroneDeploy, DroneLogbook, etc.) | Skydio Cloud is excellent for Skydio-only ops. Mixed fleets (Skydio + DJI + Parrot + ANYmal ground patrols) need a single pane that nobody ships. |
| **[Anduril — Ghost / Ghost-X + Lattice OS](https://www.anduril.com/lattice/lattice-sdk)** | Drone (defense + critical infrastructure) | Lattice SDK (REST + gRPC); open data models; sandbox env; Python `anduril-lattice-sdk` ([source](https://developer.anduril.com/)) | Public developer program with qualification gate ([source](https://developer.anduril.com/reference/overview/overview)) | Lattice OS (in-house tactical UI) | Lattice is military-grade. Adjacent civilian/critical-infra customers (utility, port, border) want similar fusion UIs without defense-procurement complexity. |
| **[Parrot — Anafi Ai / Anafi USA](https://www.parrot.com/)** | Drone (commercial inspection, NDAA-compliant alt) | Olympe SDK (Python); GroundSDK (iOS/Android); MAVLink-compatible ([source](https://mavlink.io/en/)) | Fully public SDKs | Parrot FreeFlight 7 (mobile); third-party AirData, Skyward, etc. | Branded enterprise fleet UI is uncovered — Parrot itself ships only mobile controller. |

### 1.6 Service — Indoor service / hospitality / healthcare

| Vendor + product | Class | Command API | SDK public? | Operator UI today | Notable gap |
|---|---|---|---|---|---|
| **[Diligent Robotics — Moxi / Moxi 2.0](https://www.diligentrobots.com/moxi)** | Service (hospital logistics, mobile manipulator) | Internal API; integrates with hospital EHR / inventory systems; robot-as-a-service model ([source](https://www.diligentrobots.com/blog/moxi2-0)) | Not public; partner integrations only | Hospital-facing console (in-house); 25+ US hospitals deployed | Hospital IT teams want a vendor-neutral dashboard that shows Moxi + Aethon TUG + Aethon HOMER on the same screen. Nobody ships this. |
| **[Aethon — TUG](https://aethon.com/)** | Service (hospital logistics, autonomous cart) | Proprietary navigation + mission system; lockable secure compartments; integrates with hospital elevators ([source](https://aethon.com/hospital-robots-healthcare/)) | Not public | Aethon-managed support center UI | TUG is a 2010s-era platform with 2010s-era UI. Hospitals running 37+ VA installations want a modern web surface. |
| **[Avidbots — Neo 2 / Kas (Command Center)](https://avidbots.com/)** | Service (autonomous floor scrubber) | Internal API; mobile-alert webhooks; cleaning-operation reports ([source](https://avidbots.com/resources/blog/avidbots-blog-neo-2-the-industrial-floor-scrubber-leader-in-fleet-performance-visibility-tracking-and-management/)) | Not public; Command Center is the only operator surface | Avidbots Command Center (in-house web) — fleet status, video monitor, alerts | Facility managers want branded "your-airport / your-mall" surfaces, not Avidbots-branded ones. Companion layer that summarizes "what cleaned, what didn't, why" is uncovered. |

### 1.7 Last-mile — Sidewalk / curb delivery

| Vendor + product | Class | Command API | SDK public? | Operator UI today | Notable gap |
|---|---|---|---|---|---|
| **[Starship Technologies — Sidewalk delivery robot](https://www.starship.xyz/)** | Last-mile (sidewalk delivery) | Internal; remote-control fallback when autonomy fails ([source](https://en.wikipedia.org/wiki/Starship_Technologies)); 3000+ robots, 10M+ deliveries, 125k road crossings/day ([source](https://www.starship.xyz/press/starship-technologies-surpasses-8-million-deliveries/)) | Not public; closed B2B platform | Starship-operated teleop pool (in-house) | Restaurant / retail partners get a tracking widget only — branded operator UX for partner staff is missing. |
| **[Nuro — Nuro Driver](https://www.nuro.ai/)** | Last-mile (road delivery, AV stack licensing) | Customizable SDK for licensees; remote-assistance API ([source](https://www.nuro.ai/technology)); pivoted to autonomy licensing in Sept 2024 | Partner SDK only; licensing model | Nuro internal teleop UI; not partner-facing | OEMs licensing Nuro Driver need branded operator + remote-assist UIs; Nuro doesn't ship them. |
| **[Serve Robotics (SERV)](https://www.serverobotics.com/)** | Last-mile (sidewalk delivery, public) | NVIDIA Jetson Orin + Ouster REV7 lidar; internal autonomy ([source](https://www.nvidia.com/en-us/case-studies/serve-robotics/)); 2,000+ robots, 99.8% completion, 1M miles/month telemetry ([source](https://serverobotics.gcs-web.com/news-releases/news-release-details/serve-robotics-builds-2000-autonomous-delivery-robots-creating)) | Not public; Uber Eats + DoorDash integrations | Serve internal teleop pool | Same gap as Starship — partner staff (restaurants, city operations) need read-only branded views. |

### 1.8 Inventory summary — where the openings are

| Pattern | Vendors exhibiting it | Studio angle |
|---|---|---|
| **Open SDK, no production UI** | Unitree (G1/H1/Go2/B2), Franka, Parrot | Build the UI. Highest near-term win rate. |
| **Closed SDK, generic in-house UI ripe for white-label** | Locus, MiR, OTTO, ANYbotics, Avidbots, Aethon | Partner-led; longer cycle, higher contract size. |
| **Closed everything (don't waste cycles)** | Tesla Optimus, Nuro (pre-licensing) | Skip. |
| **Strong third-party UI already** | Spot (Foxglove + Orbit), DJI (FlightHub 2 + FlytBase), Skydio (Cloud) | Differentiate on vertical UX or AI-companion, not on raw observability. |

---

## 2. Competitive landscape — operator / observability platforms

Five direct comparators plus the open-source baseline. Studio competes against Foxglove and Formant first, InOrbit second, Encord adjacently, and rosbridge/robot_web_tools whenever a robotics team decides to roll their own.

### 2.1 [Foxglove](https://foxglove.dev/)

| Field | Detail |
|---|---|
| **Funding** | $40M Series B led by Bessemer Venture Partners, announced Nov 2025 (existing: Eclipse, Amplify). Total raised: $58M+ since 2021 ([source](https://www.businesswire.com/news/home/20251112126106/en/Foxglove-Raises-$40-Million-Series-B-to-Power-the-Future-of-Physical-AI)) |
| **Customers (named publicly)** | NVIDIA, Amazon, Anduril, Wayve, Dexterity, Chef Robotics, Shield AI ([source](https://www.therobotreport.com/foxglove-raises-40m-scale-data-platform-roboticists/)) |
| **Positioning** | "Data stack for Physical AI" — visualization (3D + video + audio + GNSS + time-series), data lake at petabyte scale, MCAP format ownership |
| **What they cover well** | (a) Rich panel-based visualization in browser + native app. (b) [MCAP](https://foxglove.dev/product/mcap) is the de facto recording format. (c) [Foxglove WebSocket protocol](https://github.com/foxglove/ws-protocol) supports JSON + Protobuf + ROS 2 .msg/.idl schemas. (d) [`foxglove_bridge`](https://docs.foxglove.dev/docs/visualization/ros-foxglove-bridge) is C++ and outperforms rosbridge on throughput. (e) Cloud storage, query, dataset comparison. |
| **What they don't cover** | (a) **Branded OEM operator surface** — Foxglove is Foxglove-branded; OEMs can't ship "your-vendor-name Console" with Foxglove on the inside. (b) **Vertical UX** — agro inspection, hospital logistics, sidewalk delivery operator workflows are not first-class; you get generic panels. (c) **AI-companion narrative layer** — no LLM-driven "explain why robot did X" view. (d) **Autonomy-fallback UX** — when robot needs teleop intervention, Foxglove's panels are not opinionated about handover. (e) **Operations workflows** — escalation, on-call rotation, runbooks. |
| **Pricing** | Free tier; team plans on request; enterprise contract |

### 2.2 [Formant](https://formant.io/)

| Field | Detail |
|---|---|
| **Funding** | ~$45M total over 3 rounds; Series B Oct 2023, $21M, led by BMW i Ventures ([Tracxn](https://tracxn.com/d/companies/formant/__57eZfBvFCaCQUMRBcO3pdriFRxYi-3vmPke--QFN7h0)); positioned as fleet-ops platform since 2017. (Corrected from an earlier ~$22M-$30M estimate — see `robot-ecosystem-roles-and-vendors.md` § 4.1.) |
| **Customers (named publicly)** | Cobalt Robotics, Built Robotics, Bear Robotics, Postmates (legacy), various pilot-stage humanoid + AMR companies |
| **Positioning** | "Data and operations platform for robotics fleets" — combines observability + teleoperation + analytics |
| **What they cover well** | (a) Out-of-the-box teleop UI (joystick-style + first-person view). (b) Fleet management (onboard, monitor, optimize). (c) [Formant Analytics](https://www.therobotreport.com/formant-launches-analytics-feature-optimize-fleet-performance/) for fleet KPIs. (d) Integrations with Slack, PagerDuty, Jira. (e) Free tier for early-stage roboticists ([source](https://www.robotics247.com/article/formant_offers_early_stage_robotics_companies_free_cloud_monitoring_platform)). |
| **What they don't cover** | (a) **Branded OEM surface** — same Foxglove problem; Formant is the brand. (b) **AI-companion narrative** — telemetry is shown raw, no LLM summarization of fleet behavior. (c) **Vertical workflows** — generic teleop, not "hospital delivery" or "vineyard inspection". (d) **Customer / end-user facing surfaces** — Formant assumes internal ops users only. |
| **Pricing** | Free tier for individuals; paid tiers undisclosed on public site as of May 2026 |

### 2.3 [InOrbit.AI](https://www.inorbit.ai/)

| Field | Detail |
|---|---|
| **Funding** | $10M Series A, Sept 2025, co-led by L'ATTITUDE Ventures + Globant Ventures ([source](https://www.businesswire.com/news/home/20250930673347/en/InOrbit.AI-Secures-Series-A-Funding-to-Scale-Robot-Orchestration-Platform-Unlocking-a-New-Era-of-Software-Driven-Physical-Workflows)) |
| **Customers (named publicly)** | Colgate-Palmolive, Genentech (Roche), Google, Qualcomm, Sprint, Brain Corp., Bossa Nova Robotics, "a dozen+ robot manufacturers" ([source](https://www.inorbit.ai/press/series-a-investment)) |
| **Positioning** | "Robot orchestration" — multi-vendor fleet coordination, [VDA 5050](https://github.com/VDA5050/VDA5050) interoperability emphasis |
| **What they cover well** | (a) Cross-vendor fleet orchestration (the strongest in this list at heterogeneous fleets). (b) Mission-level abstractions (not just teleop). (c) Enterprise sales motion proven (Colgate-Palmolive scale). (d) RobOps methodology and thought leadership. |
| **What they don't cover** | (a) **Branded OEM surface** — InOrbit-branded. (b) **AI-companion narrative**. (c) **Modern UI polish** — UX is enterprise-functional, not delightful. (d) **Self-serve developer onboarding** — sales-led. |
| **Pricing** | Enterprise; quoted ranges $50k–$500k/yr based on fleet size |

### 2.4 [Encord](https://encord.com/)

| Field | Detail |
|---|---|
| **Funding** | $60M Feb 2026 (latest round) — physical-AI data infrastructure ([source](https://siliconangle.com/2026/02/26/physical-ai-data-infrastructure-startup-encord-lands-60m-accelerate-intelligent-robot-drone-development/)) |
| **Customers (named publicly)** | Toyota Woven, Zipline, Skydio, 300+ physical-AI teams ([source](https://encord.com/blog/best-data-annotation-tools-for-physical-ai/)) |
| **Positioning** | "Train and run AI on the right data" — annotation, curation, model evaluation for physical-AI teams |
| **What they cover well** | (a) Multimodal annotation (3D LiDAR, point clouds, camera, telemetry). (b) Data curation + model evaluation against ground truth. (c) HITL workflows. (d) [Physical AI Suite](https://encord.com/blog/physical-ai-suite/) launched 2025. |
| **What they don't cover** | (a) **Live operations** — Encord is build-time, not run-time. (b) **Operator UI / teleop** — out of scope. (c) **Fleet management**. (d) **AI-companion at point of operation**. |
| **Why they matter to studio** | Adjacent rather than competitive. Many of studio's prospective customers will already be Encord users for data ops; the run-time operator UI is a separate slot. |

### 2.5 Open-source baselines

| Project | What it is | What it lacks |
|---|---|---|
| **[rosbridge_suite](https://github.com/RobotWebTools/rosbridge_suite)** | JSON-over-WebSocket bridge from ROS 1 / ROS 2 to web clients. Server in Python. | Performance on high-frequency topics (Foxglove explicitly recommends against it now — [source](https://foxglove.dev/blog/using-rosbridge-with-ros2)); no parameter introspection; no graph view; no auth model fit for SaaS. |
| **[robot_web_tools (roslibjs, ros3djs, mjpegcanvasjs)](http://robotwebtools.org/)** | JavaScript libraries for ROS topic pub/sub, 3D scene, MJPEG video in browser. | DIY assembly; no opinionated operator UX; no fleet model; no AI companion; security and TLS termination are your problem. |
| **[Foxglove Bridge](https://github.com/foxglove/ros-foxglove-bridge)** | C++ WebSocket server with the Foxglove protocol over ROS 1/2; high-performance. | Server only. No UI. Pair with custom front-end or with Foxglove client. |
| **[sciota-robotics](https://github.com/sciota-robotics)** (and similar boutique frameworks) | Small-scope open-source robot dashboards from university labs. | Maintenance discontinuous; not production-grade. |
| **[Open-RMF](https://github.com/open-rmf)** | Open Robotics Middleware Framework — cross-fleet coordination from Open Robotics + Intrinsic | Server-side only; needs a UI; adoption concentrated in research/airports. |

### 2.6 Side-by-side gap matrix

| Capability | Foxglove | Formant | InOrbit | Encord | Open-source | **Studio target** |
|---|---|---|---|---|---|---|
| Live telemetry visualization | ✅ Strong | ✅ Good | ✅ OK | ❌ N/A | ⚠️ DIY | ✅ Required (via Foxglove or roslibjs) |
| Teleop UI | ⚠️ Limited | ✅ Strong | ✅ OK | ❌ | ⚠️ DIY | ⚠️ Optional |
| Branded white-label OEM surface | ❌ Foxglove-branded | ❌ Formant-branded | ❌ InOrbit-branded | ❌ | ✅ DIY (your brand) | ✅ **Core differentiator** |
| Vertical-specific UX (hospital, agro, etc.) | ❌ Generic | ❌ Generic | ⚠️ Some via partners | ❌ | ⚠️ DIY | ✅ **Core differentiator** |
| AI-companion narrative layer (LLM-driven) | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ **Core differentiator** |
| Autonomy-fallback handover UX | ⚠️ | ✅ Good | ⚠️ | ❌ | ❌ | ✅ Required |
| Multi-vendor fleet orchestration | ❌ | ⚠️ | ✅ Strong | ❌ | ⚠️ (Open-RMF) | ⚠️ Phase 2 |
| Data labeling / model eval | ⚠️ | ❌ | ❌ | ✅ Strong | ❌ | ❌ Out of scope |
| Compliance + audit log surfaces (regulated verticals) | ❌ | ⚠️ | ⚠️ | ❌ | ❌ | ✅ Required for medical / defense / agro |

The interesting columns are the three with no incumbent: branded OEM surface, vertical UX, AI-companion. Studio's defensible angle is the intersection of those three.

---

## 3. Integration protocols / transports — what the Gateway layer must speak

A practical Gateway for studio's operator-surface vertical must speak at least 5-7 of the following. None is universal; each dominates in one or two robot classes.

| Protocol | Transport | Ecosystem maturity | Web compatibility | Where it dominates | Notes |
|---|---|---|---|---|---|
| **[ROS 2 + Foxglove Bridge](https://github.com/foxglove/ros-foxglove-bridge)** | WebSocket (C++ server) | Very mature in research; rising in production. ROS 2 Jazzy is the LTS as of 2026. | Native browser; JSON + Protobuf + ROS 2 .msg/.idl ([source](https://foxglove.dev/blog/announcing-the-foxglove-websocket-protocol)) | Humanoid R&D, quadrupeds, university research, most US/EU startup robotics | Studio default. Better than rosbridge for any new project. |
| **[rosbridge_suite (JSON-over-WS)](https://github.com/RobotWebTools/rosbridge_suite)** | WebSocket (Python server) | Mature; declining recommendation | Native browser; JSON only via roslibjs | Legacy ROS 1 deployments; sidewalk delivery testbeds | Support for back-compat but lead with Foxglove Bridge. |
| **[Foxglove WebSocket protocol + MCAP recordings](https://foxglove.dev/product/mcap)** | WebSocket live + file format for storage | Mature 2.0+; Bessemer-backed | First-class — Foxglove SDK in TS/Rust/Python | Any Foxglove-using customer | MCAP is the right recording format full stop. Studio should record everything as MCAP. |
| **[VDA 5050 (JSON over MQTT)](https://github.com/VDA5050/VDA5050)** | MQTT (typically [HiveMQ](https://www.hivemq.com/) or [EMQX](https://www.emqx.com/)) | Mature in EU AMR; v3.0 published March 2026 ([source](https://bluebotics.com/vda-5050-explained-agv-communication-standard/)) | Browser via MQTT-over-WebSocket | EU/IL AMR fleets (logistics, intralogistics) | **Critical for the AMR ICP**. Studio Gateway must speak VDA 5050 v3.0 to be credible with Locus/MiR/OTTO/Geek+ customers. |
| **[MQTT (general)](https://mqtt.org/)** | TCP + WebSocket variant | Very mature (15+ years) | Native via MQTT-over-WebSocket; libraries everywhere | DJI Cloud API, IoT telemetry, fleet telematics, VDA 5050 underlying ([source](https://www.emqx.com/en/solutions/fleet-telematics)) | Pair with Cloudflare Workers via Hyperdrive-style edge proxy or with HiveMQ/EMQX broker. |
| **[Spot SDK (gRPC over TLS)](https://dev.bostondynamics.com/docs/protos/README.html)** | gRPC | Mature; well-documented; weekly releases | Not browser-native (needs gRPC-Web proxy or BFF) | Boston Dynamics Spot fleets | Studio Gateway needs a Spot proxy translating gRPC ↔ WebSocket. |
| **[MAVLink](https://mavlink.io/en/)** | UDP/TCP/serial; MAVLink2 wire format | Very mature in drone ecosystem (since 2009) | Via Mavelous-style or `mavsdk` gateways; can be tunneled over WebSocket | Drone fleets (Parrot, ArduPilot/PX4-based) | Used by QGroundControl, Mavelous, ADOSMissionControl ([source](https://github.com/altnautica/ADOSMissionControl)). MAVLink2 has 14-byte overhead. |
| **[Lattice SDK (REST + gRPC)](https://developer.anduril.com/)** | HTTPS + gRPC | Newer (2023+); defense-grade | REST is browser-native; gRPC needs proxy | Anduril Lattice deployments (defense, critical infra) | Studio touch-point only if pursuing defense-adjacent customers. |
| **[URScript / Universal Robots REST + PolyScope X SDK](https://docs.universal-robots.com/PolyScopeX_SDK_Documentation/build/SDK-v0.18/Resources.html)** | TCP for URScript; REST for PolyScope X | Mature; large UR+ ecosystem | REST is browser-native; URScript is socket-level | UR cobot fleets in manufacturing | Studio Gateway needs HTTP REST adapter + URCap-hosted endpoints. |
| **[ABB Robot Web Services (REST + WebSocket)](https://developercenter.robotstudio.com/api/pcsdk/)** | HTTPS + WebSocket | Mature; OmniCore is the new default | Native browser | ABB IRC5 (sunsetting June 2026) + OmniCore | Direct fit; minimal adapter work. |
| **[DJI Cloud API (MQTT + HTTPS + WebSocket)](https://developer.dji.com/doc/cloud-api-tutorial/en/)** | MQTT + HTTPS + WS | Mature; v3+ in 2026 | Native | DJI Dock 3 / Matrice 4D deployments | Pair with VDA 5050 stack — same MQTT broker can multiplex. |
| **[OPC UA](https://opcfoundation.org/about/opc-technologies/opc-ua/)** | TCP-binary or HTTPS | Very mature in industrial automation | Via UA-over-WS gateway | ABB IoT Gateway, KUKA-via-iiQKA, factory floor integration | Important for "talk to MES/SCADA from operator dashboard" use cases. |

### 3.1 Minimum viable Gateway scope

For the studio's first 3-5 paid pilots, the realistic protocol coverage is:

1. **Foxglove Bridge / WebSocket protocol + MCAP** (ROS 2 + research-leaning customers)
2. **VDA 5050 over MQTT** (AMR logistics customers)
3. **MAVLink** (drone customers, civilian)
4. **DJI Cloud API** (DJI-fleet customers wanting branded alternative to FlightHub 2)
5. **Spot SDK gRPC** (quadruped customers)
6. **HTTPS REST** (URScript REST, ABB RWS, ANYmal API) — single adapter wraps all three with per-vendor schemas

Everything else (KUKA KRL, Lattice, Lutron-style proprietary) on demand.

---

## 4. ICP segments — three reachable bets

Studio is small, Moldova-based, and competes for trust before competing for features. The three ICPs below are ordered by reachability: warmest first.

### 4.1 ICP A — Pre-Series-B humanoid startups in EU / IL needing operator-companion UI

| Field | Detail |
|---|---|
| **Who buys** | CTO or VP Engineering at humanoid startup, ~10-50 engineers, $5M-$30M total raised. Geographies: Germany (Neura, Sanctuary-EU footprints, NEXTGEN ROBOTICS), UK (Humanoid Labs), Switzerland (LEAP), Israel (Mentee Robotics, Menteebot), Estonia (Starship's adjacent talent pool). |
| **Pain they pay for** | (a) Demo'ing to enterprise customers requires a polished operator UI; Foxglove looks "internal-tool-y" and is Foxglove-branded. (b) Helix/VLA-model startups need to demonstrate that an operator can intervene safely — no incumbent ships handover UX. (c) Fundraising decks require "operator experience" video — a branded surface dramatically improves the demo. |
| **Price band** | $80k–$200k pilot (12-week scoped engagement). Recurring $4k–$15k/mo SaaS once pilot ships. |
| **Why studio wins vs Foxglove** | Foxglove won't white-label. Studio ships **your-brand operator console** on Foxglove Bridge underneath. Studio's Muse Architect layer narrates "what the robot just did and why" in the operator's language — Foxglove has nothing here. |
| **Reachability** | High. EU/IL ecosystem overlaps with Pavel's existing network; humanoid CTOs are reading the same papers and Twitter feeds. |
| **First 3 names to approach** | Mentee Robotics (Tel Aviv), Neura Robotics (Munich), Humanoid (London). |

### 4.2 ICP B — AMR fleet operators in logistics with VDA 5050 + branded white-label dashboard

| Field | Detail |
|---|---|
| **Who buys** | Head of Automation or COO at a 3PL / logistics operator running mixed-vendor AMR fleets in EU. Companies in the 200-2000-employee range; warehouses 20k-200k m². Geographies: Germany, Netherlands, Czechia, Romania, Poland, Israel. |
| **Pain they pay for** | (a) They run Locus + MiR + Geek+ + an OTTO or two; no single pane. (b) MiR Fleet shows 100 MiRs; Locus Server shows 200 Locus units; nobody sees the floor. (c) They have to relabel everything in Bulgarian or Romanian and incumbents won't. (d) Executive dashboards (cycle time per SKU, robot utilization vs labor) don't exist out of the box. |
| **Price band** | $50k–$150k pilot. $3k–$10k/mo SaaS after. |
| **Why studio wins vs InOrbit** | InOrbit ships VDA 5050 orchestration too, but their UX is dated and their pricing is enterprise-only. Studio undercuts on price (small operator, low overhead) and over-delivers on UX polish + localization (RO/RU/HE) + AI companion that pulls fleet anomalies into a daily digest. |
| **Reachability** | Medium-high. Moldova-adjacent logistics market in RO/PL/HU/CZ is reachable via partner SIs and warm intros; VDA 5050 customer councils are small. |
| **First 3 names to approach** | Damen Shipyards Romania, Cargus, Dr. Max Pharmacy logistics network. |

### 4.3 ICP C — Vertical-specific OEM (agro, cleaning, medical, defense-adjacent) with custom branded operator surface + AI post-mortem

| Field | Detail |
|---|---|
| **Who buys** | Founder/CTO at a vertical robot OEM, often $2M-$20M raised, often building on Unitree or ANYmal hardware, sometimes designing custom platforms. Verticals: precision agriculture (vineyards, orchards), industrial cleaning, hospital logistics (Moxi-adjacent), defense-adjacent civilian (port patrol, perimeter security). Geographies: Israel (very strong in agro, defense-adjacent), Spain/Italy (agro), Germany (cleaning, medical), Romania/Moldova (agro). |
| **Pain they pay for** | (a) They're a small team; building a polished operator surface takes 6-12 months they don't have. (b) Their customer-facing UI is what their enterprise buyers actually see, not their AI stack — so UI quality is a sales lever. (c) Post-mission AI analysis ("Tell me what happened during today's 4-hour run, in one paragraph") is what their non-technical end users actually want, and nobody ships it. |
| **Price band** | $30k–$120k for an initial branded shell + 1 integration. $2k–$8k/mo SaaS. Larger projects ($150k-$300k) when wrapping the whole product. |
| **Why studio wins** | Foxglove and Formant are horizontal — they don't speak "vineyard inspection" or "ICU room transit". Studio's verticalization (operator screens that look like the customer's industry, not like a ROS panel) plus the AI companion (daily/weekly narrative for the customer's customer) is a fit no incumbent currently competes for. |
| **Reachability** | Medium. Agro and defense-adjacent in IL/ES/IT need warm intros; Moldova/Romania agro is direct. |
| **First 3 names to approach** | Tevel Aerobotics (IL, agro drones), Naïo Technologies (FR, agro AMR), MagosLab (defense-adjacent ground patrol, IL). |

---

## 5. Gaps where studio has a defensible angle

Five concrete gaps that emerged from sections 1-4.

### 5.1 Branded OEM operator surfaces with shared substrate

**The gap.** Foxglove, Formant, and InOrbit all force the OEM to display their brand. OEMs selling to enterprises (hospitals, refineries, ports, factories) need to ship a surface that says **their** name, not "Powered by Foxglove."

**Why incumbents don't fill it.** Foxglove and Formant raised on a thesis that says they ARE the operator UI. White-labeling breaks their go-to-market — every white-label deal is a deal they don't get a brand impression from. They will not pivot.

**Why studio fits.** Studio's business model is bespoke + SaaS-recurring; white-labeling is the norm, not the exception. Stack (Next.js + Tailwind + design tokens) supports re-theming in hours, not weeks. Studio's revenue scales per OEM, not per end-user, so margin model accommodates white-label without cannibalizing.

### 5.2 AI-companion narrative layer on top of telemetry

**The gap.** Every operator surface today shows raw telemetry — graphs, 3D scenes, log lines. Nobody narrates it. A floor manager asking "what happened during the second shift?" gets a list of timestamps. A hospital nurse asking "did Moxi finish the meds round?" gets a JSON event.

**Why incumbents don't fill it.** Foxglove and Formant are observability companies; LLMs are not their core competency. They will eventually bolt on summaries (Foxglove's late-2025 announcements gesture at this), but it's not their identity. Their existing customers (NVIDIA, Anduril) don't need it — those customers already have ML teams.

**Why studio fits.** Studio's Muse Architect IP is precisely an LLM-companion-on-top-of-structured-state thesis. The same primitives that explain a codebase to a developer can explain a robot run to an operator. Studio's Anthropic Claude integration is already production-grade. The competitive moat is product taste, not technology.

### 5.3 Autonomy-fallback / handover UX

**The gap.** Sidewalk-delivery robots (Starship, Serve, Nuro), humanoids (Figure, Apptronik, 1X), and inspection quadrupeds all need a "robot gives up, human takes over" UX. The handover is hard: the human needs context fast (last 30 seconds of video, what the robot was trying to do, what blocked it) and needs control modalities that match the situation (full teleop, guided suggestion, "approve this plan"). No incumbent has a strong opinion here. Formant's teleop is generic. Foxglove panels are not workflow-aware.

**Why incumbents don't fill it.** It's verb-shaped work (workflow, escalation, decision support) not noun-shaped work (data, visualization). Foxglove/Formant are noun-shaped companies.

**Why studio fits.** Handover UX is the same shape as "AI agent asks human for help" — a problem the studio is already solving for Maestro/Muse. The mental model maps cleanly.

### 5.4 Vertical-specific UX for non-roboticist end users

**The gap.** A vineyard manager doesn't want to see ROS topics. A hospital pharmacy lead doesn't want to see MQTT payloads. They want screens that look like their industry — schedules, completion rates, exception queues, photos of what got picked. Today every OEM rebuilds these from scratch on top of Foxglove/Formant because the horizontal tools don't ship vertical screens.

**Why incumbents don't fill it.** Horizontal scale economics. Foxglove has 10,000+ developers — if they ship a "vineyard panel" they're investing in a niche.

**Why studio fits.** Studio's natural economic model is vertical: each engagement is a different vertical, repeatable UX patterns get harvested into the substrate. Studio's Next.js/Tailwind/MDX stack is built for fast vertical iteration. Studio's translation infrastructure (`@rapoport-studio/i18n`) handles RO/RU/HE out of the box — useful in EU/IL agro and medical.

### 5.5 Compliance + audit trail surfaces for regulated verticals

**The gap.** Medical (HIPAA in US, MDR in EU), defense (NDAA, ITAR), agro/food (FDA/EFSA traceability) all need audit log surfaces that prove "who saw this robot output, when, what they did." Foxglove and Formant log internally but don't ship the audit surface. InOrbit gets closer (enterprise-grade) but is dated.

**Why incumbents don't fill it.** Slow customer cycle, region-specific (CNIL vs HIPAA vs MDR each demand different shapes). Doesn't fit horizontal economics.

**Why studio fits.** Studio is already building cookie-law / GDPR / MDR surfaces for adjacent work (`cookie-law-jurisdictions.md`, RAP-828 harden-cookie-transparency). The same audit-log primitive — append-only event store, with operator-facing review surface — is reusable for robotics compliance.

---

## 6. Risks & open questions

Honest risk list. The vertical is real but studio is small. Each item below should be a yellow flag during early conversations.

| # | Risk / open question | Severity | Mitigation hypothesis |
|---|---|---|---|
| 1 | **ROS 2 expertise gap inside studio.** Pavel and the studio team are web/AI-native; ROS 2 has its own culture, debugging tools, and gotchas (DDS discovery, QoS profiles, transient-local durability). Customer credibility evaporates if studio engineers can't debug a topic discovery problem. | **High** | Hire or contract one senior ROS 2 engineer per active engagement. Don't bluff. Studio's value is the surface; the ROS plumbing is the customer's responsibility unless explicitly scoped. |
| 2 | **Sales cycle.** Robotics OEM contracts run 3-9 months from first call to signature. Studio's monthly burn requires shorter cycles than that. First 3 customers will probably be warm-intro pilots, not cold outbound. | **High** | Anchor on existing network (RAP-826 contract page warm pipeline). Sell pilots at $30k-$80k that close in <8 weeks. Repeat. |
| 3 | **Geographic disadvantage — Moldova address on the invoice.** Western enterprise procurement teams (especially defense-adjacent and hospital-adjacent) flag non-EU/non-US vendors. SRL legitimacy + EU/IL local entity may be required for some deals. | **Medium-high** | Lead with EU pilots (Romania, Germany, Czech, Netherlands). Establish a billing relationship via an EU subsidiary or partner-of-record for procurement-sensitive deals. Israel is friendly to small foreign vendors. |
| 4 | **Hardware-adjacent test requirements.** Studio cannot afford a Spot ($75k+), Apollo (~$150k+ list), ANYmal (~$150k+). Without hardware in the office, debugging integration issues becomes hard. | **Medium-high** | (a) Buy a single Unitree Go2 ($1.6k-$3k) as a known-good test rig. (b) Use Gazebo + ROS 2 simulation for everything else. (c) Require customers to provide either remote access to a robot or recorded MCAP files for development. |
| 5 | **"AI companion" trust calibration.** LLMs hallucinate. A robotics operator UI that says "Robot 12 has finished the medication round" when it actually got stuck halfway is worse than no companion at all. Liability in medical / defense is real. | **High** | Companion layer must be retrieval-only over structured state (not free-form generation). Citations to source events, with timestamp ranges, always visible. Hard rule: the companion can describe what the structured event store says, never what it "thinks happened." |
| 6 | **VDA 5050 evolution.** Spec moved v2.0 → v2.1 → v3.0 in two years. Studio's adapter is a maintenance burden, especially as customers want a mix of v2 and v3 fleets ([source](https://bluebotics.com/vda-5050-explained-agv-communication-standard/)). | **Medium** | Treat VDA 5050 as a stable adapter inside packages/gateway; version-pin per customer; budget 1 dev-week per minor version bump. |
| 7 | **Foxglove (or Formant) ships AI companion.** Bessemer-backed Foxglove has the capital and the Anthropic relationships to ship "Foxglove AI" within 6-12 months. The differentiation evaporates. | **Medium** | Studio's defense is not the AI feature itself; it's (a) branded surface, (b) vertical UX, (c) project-shape (bespoke + ongoing). Foxglove will never out-bespoke a boutique studio. Stay on the project side of the buy/build line. |
| 8 | **Customer-pulled scope creep into autonomy itself.** Robotics customers will try to push studio from "operator UI" into "navigation policy" or "manipulation planning". That's not the studio's competency and the engagements will fail. | **Medium** | Hard line in proposal contracts: studio delivers operator surface, telemetry, companion, audit. Studio does NOT deliver autonomy, control law, perception. Refer integrations to ROS 2 consultants. |
| 9 | **Open question — should studio productize?** The deeper into this vertical studio gets, the more it looks like a productized "Foxglove-but-white-label." Whether to keep this as services-led + reusable substrate, or pivot to product, is unresolved. | Open | Decide after 5 closed pilots. Until then: services-led with substrate harvest. |
| 10 | **Open question — defense exposure.** Anduril/Skydio/Shield AI are the most aggressive buyers but their procurement is hostile to small foreign vendors. Should studio pursue defense-adjacent at all? | Open | Decline until studio has an EU/US entity with the right clearances. Pursue civilian critical-infrastructure (utilities, ports) instead — same protocols, far easier procurement. |
| 11 | **Open question — Unitree dependence.** Unitree is the most attractive entry point (open SDK, no UI, exploding install base) but the company is Chinese; some EU/US customers will reject anything that touches Unitree firmware in 12-24 months. | Open | Build the substrate to be Unitree-friendly but not Unitree-locked. ANYbotics/Spot/Apptronik are the path-of-escape if Unitree becomes politically inviable. |

---

## Sources & primary references

### Operator/observability platforms
- [Foxglove $40M Series B (Bessemer-led, Nov 2025)](https://www.businesswire.com/news/home/20251112126106/en/Foxglove-Raises-$40-Million-Series-B-to-Power-the-Future-of-Physical-AI)
- [The Robot Report — Foxglove raises $40M](https://www.therobotreport.com/foxglove-raises-40m-scale-data-platform-roboticists/)
- [Foxglove WebSocket protocol announcement](https://foxglove.dev/blog/announcing-the-foxglove-websocket-protocol)
- [foxglove/ros-foxglove-bridge GitHub](https://github.com/foxglove/ros-foxglove-bridge)
- [foxglove/ws-protocol spec](https://github.com/foxglove/ws-protocol/blob/main/docs/spec.md)
- [MCAP file format spec](https://foxglove.dev/product/mcap)
- [Formant fleet management product](https://formant.io/)
- [Formant Analytics launch](https://www.therobotreport.com/formant-launches-analytics-feature-optimize-fleet-performance/)
- [Formant free tier announcement](https://www.robotics247.com/article/formant_offers_early_stage_robotics_companies_free_cloud_monitoring_platform)
- [InOrbit.AI Series A — Globant + L'ATTITUDE Ventures (Sept 2025)](https://www.businesswire.com/news/home/20250930673347/en/InOrbit.AI-Secures-Series-A-Funding-to-Scale-Robot-Orchestration-Platform-Unlocking-a-New-Era-of-Software-Driven-Physical-Workflows)
- [InOrbit press release on Series A](https://www.inorbit.ai/press/series-a-investment)
- [Encord $60M (Feb 2026) — Physical AI](https://siliconangle.com/2026/02/26/physical-ai-data-infrastructure-startup-encord-lands-60m-accelerate-intelligent-robot-drone-development/)
- [Encord Physical AI Suite (LiDAR, point cloud)](https://encord.com/blog/physical-ai-suite/)
- [rosbridge_suite GitHub](https://github.com/RobotWebTools/rosbridge_suite)
- [Robot Web Tools](http://robotwebtools.org/)
- [Foxglove blog — using rosbridge with ROS 2 (recommends Foxglove Bridge)](https://foxglove.dev/blog/using-rosbridge-with-ros2)

### Robot vendors & SDKs
- [Boston Dynamics Spot SDK](https://dev.bostondynamics.com/) / [GitHub](https://github.com/boston-dynamics/spot-sdk)
- [Universal Robots PolyScope X SDK](https://docs.universal-robots.com/PolyScopeX_SDK_Documentation/build/SDK-v0.18/Resources.html)
- [Franka Robotics development](https://franka.de/develop) / [GitHub](https://github.com/frankaemika)
- [ABB OmniCore](https://www.abb.com/global/en/areas/robotics/products/controllers/omnicore) / [Developer Center](https://developercenter.robotstudio.com/api/pcsdk/)
- [KUKA System Software](https://www.kuka.com/en-us/products/robotics-systems/software/system-software/kuka_systemsoftware)
- [Unitree G1 SDK Developer Guide](https://support.unitree.com/home/en/G1_developer) / [GitHub](https://github.com/unitreerobotics)
- [ANYbotics ANYmal](https://www.anybotics.com/robotics/anymal/) / [Siemens developer portal entry](https://developer.siemens.com/anybotics/overview.html)
- [DJI Developer](https://developer.dji.com/) / [Cloud API tutorial](https://developer.dji.com/doc/cloud-api-tutorial/en/)
- [Skydio Cloud API](https://apidocs.skydio.com/reference/introduction) / [Developer tools](https://www.skydio.com/developer-tools)
- [Anduril Lattice SDK](https://developer.anduril.com/) / [Reference](https://developer.anduril.com/reference/overview/overview)
- [Figure AI](https://www.figure.ai/) / [Figure 02 specs](https://humanoidspecs.com/robots/figure-02)
- [Apptronik Apollo](https://apptronik.com/apollo)
- [1X Neo](https://www.1x.tech/) / [Specs writeup](https://parametric-architecture.com/neo-by-1x-worlds-first-humanoid-robot/)
- [Tesla AI & Robotics](https://www.tesla.com/AI)
- [Locus Robotics — Series F](https://www.prnewswire.com/news-releases/locus-robotics-announces-117-million-in-series-f-funding-bringing-its-valuation-close-to-2-billion-301688564.html)
- [MiR Fleet Enterprise](https://mobile-industrial-robots.com/products/software/mir-fleet)
- [OTTO by Rockwell Automation](https://ottomotors.com/)
- [Geek+ market share announcement](https://www.geekplus.com/resources/news/global-warehouse-automation-surges-as-geekplus-extends-no.1-amr-leadership-for-seven-consecutive-years)
- [Fetch Robotics acquisition by Zebra](https://www.zebra.com/us/en/about-zebra/newsroom/press-releases/2021/zebra-technologies-to-acquire-fetch-robotics.html) / [Wind-down coverage](https://www.therobotreport.com/zebra-technologies-winding-down-fetch-based-mobile-robot-group/)
- [Diligent Robotics Moxi 2.0](https://www.diligentrobots.com/blog/moxi2-0)
- [Aethon TUG](https://aethon.com/hospital-robots-healthcare/)
- [Avidbots Neo 2 / Command Center](https://avidbots.com/resources/blog/avidbots-blog-neo-2-the-industrial-floor-scrubber-leader-in-fleet-performance-visibility-tracking-and-management/)
- [Starship Technologies — 10M deliveries](https://www.robotics247.com/article/starship-technologies-surpasses-10-million-autonomous-deliveries)
- [Nuro technology](https://www.nuro.ai/technology)
- [Serve Robotics — 2,000+ fleet](https://serverobotics.gcs-web.com/news-releases/news-release-details/serve-robotics-builds-2000-autonomous-delivery-robots-creating)
- [Parrot drone — MAVLink-compatible](https://www.parrot.com/)

### Protocols
- [VDA 5050 GitHub](https://github.com/VDA5050/VDA5050) / [BlueBotics overview](https://bluebotics.com/vda-5050-explained-agv-communication-standard/)
- [MAVLink developer guide](https://mavlink.io/en/)
- [MQTT for fleet telemetry — EMQX](https://www.emqx.com/en/solutions/fleet-telematics) / [HiveMQ](https://www.hivemq.com/blog/empowering-real-world-iot-with-mqtt-for-fleet-tracking/)
- [QGroundControl](https://qgroundcontrol.com/)
- [OPC Foundation OPC UA](https://opcfoundation.org/about/opc-technologies/opc-ua/)
- [Open-RMF GitHub](https://github.com/open-rmf)

### Teleoperation research (for §5.3 handover UX)
- [Intuitive GUI for Non-Expert Teleop (arxiv 2510.13594)](https://arxiv.org/html/2510.13594v1)
- [Labellerr — Top teleoperation service providers 2026](https://www.labellerr.com/blog/top-teleoperation-companies-humanoid-robotics/)
- [Unitree xr_teleoperate GitHub](https://github.com/unitreerobotics/xr_teleoperate)

### Internal cross-references
- `openspec/_methodology/research/competitor-landscape-europe.md` — adjacent EU competitor scan
- `openspec/_methodology/research/competitor-landscape-israel.md` — IL competitor scan
- `openspec/_methodology/research/cookie-law-jurisdictions.md` — compliance substrate for §5.5
- `openspec/current/platform.md` — studio's substrate and stack
