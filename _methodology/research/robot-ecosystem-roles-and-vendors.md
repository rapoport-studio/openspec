# Robot Ecosystem — Roles and Vendors

> Reference document. Compiled May 2026. Scope: the **people** who build robots and the **vendors** who sell them tooling. Third companion to `robot-operator-interfaces.md` (platforms, transports, operator-UI vendors) and `robot-developer-pain-points-and-openspec.md` (where spec-driven dev fits). Where the first two docs scanned hardware and protocols, this one scans org charts and the supplier landscape — because the unit of a unified methodology is not a robot, it is a *person handing an artefact to another person*.
>
> Strategic question behind the doc: the studio is designing a spec-driven (OpenSpec) methodology that would put adapter writers, behaviour engineers, UX designers and control engineers on one shared modelling language for tasks, missions and sequences. To know whether that is plausible you have to know (a) who these people actually are, (b) what they each call "the work," and (c) whether the vocabularies are genuinely incommensurable or merely dialects. Sections 1–4 are the census. Section 5 is the Tower of Babel diagnosis. Section 6 is the honest assessment of whether one artefact can plausibly unify them.

---

## 0. Executive summary

- A robotics software + UX program is **not one discipline**. It is roughly **twelve** specialties, and they do not share a definition of "done." The behaviour engineer's done is "the tree ticks `SUCCESS` in sim"; the controls engineer's done is "the step response settles inside 2%"; the UX designer's done is "five operators completed the task unaided"; the safety engineer's done is "the hazard log is closed and the PL rating is signed." These are not the same event.
- **"Adapter writer" is not a job title.** Nobody has it on a business card. The work — building the driver/bridge/HAL layer between a robot and everything else — is *diffuse*, spread across integration engineers, embedded engineers, "robot integrations" software engineers, and systems-integrator (SI) firms. That diffuseness is itself a finding: the layer the studio's Gateway competes in has **no professional identity to anchor to**, which is both an opportunity (no incumbent guild) and a risk (no buyer who self-identifies as the customer).
- **Robotics UX is a real but tiny discipline.** It is recognised enough that Amazon, Boston Dynamics and surgical-robotics firms staff dedicated "UX Designer, Robotics" and "HRI Researcher" roles ([Amazon](https://www.amazon.jobs/en/jobs/3172158/senior-ux-designer-amazon-industrial-robotics)), and a handful of agencies (argodesign, IIIMPACT, Punchcut, Boston Engineering) sell it — but it is a rounding error next to general product UX. argodesign shaping the *physical form* of the Apptronik Apollo humanoid ([Fast Company](https://www.fastcompany.com/90239194/argodesign-sells-to-a-22-billion-it-company)) is the high-water mark of the field, and it is industrial design, not screen UX.
- **Biggest ≠ best in operator tooling.** By developer reach and capital, **Foxglove** is the biggest pure-play robotics-data/visualization vendor ($40M Series B, Bessemer-led, Nov 2025; "tens of thousands of developers" — [BusinessWire](https://www.businesswire.com/news/home/20251112126106/en/Foxglove-Raises-$40-Million-Series-B-to-Power-the-Future-of-Physical-AI)). By *operations completeness* (teleop + fleet + analytics in one product) **Formant** and **InOrbit** are arguably better for a fleet operator. By enterprise installed base inside one OEM's ecosystem, **Boston Dynamics Orbit** dwarfs all of them — but it is single-vendor. None of them white-label, and none narrate.
- **The Tower of Babel is real and it is the studio's opening.** Twelve roles, twelve file formats, twelve review processes, twelve definitions of done. Every prior attempt to unify them — MBSE/SysML, ROS REPs, MassRobotics/VDA 5050, digital-twin platforms — solved *one seam* and left the rest. A lightweight markdown-spec methodology will not be the universal modelling language either. But it could plausibly be the **shared review surface** — the one artefact every role reads even though each still works in their own tool. That, and not "one model to rule them all," is the defensible claim.

---

## 1. The roles — who builds robots

A robotics program staffs, at full scale, around a dozen distinct specialties. Smaller teams collapse several into one person (the classic "ROS generalist who also does the dashboard"), but the *artefacts* stay distinct even when the headcount doesn't. The table is the catalogue; the prose under it adds vocabulary and the in-house/contract pattern.

Salary bands below are US-market signals (2026) and should be read as *seniority proxies*, not as what a Moldova-based studio would pay or charge. They matter because they tell you which roles are scarce (high band, hard to hire → customers will *contract* them) versus abundant (lower band, in-house default).

| Role | What they do | Primary artefact they produce | Tools / languages | Native vocabulary | US salary band (2026) | In-house vs contracted |
|---|---|---|---|---|---|---|
| **Robotics software engineer / ROS developer** | Build the robot's application software — nodes, the topic graph, lifecycle, the glue between perception, planning and control | ROS 2 packages, launch files, the node graph | C++, Python, ROS 2 (Jazzy LTS), `colcon`, `rclcpp`/`rclpy`, DDS | "node," "topic," "QoS profile," "lifecycle," "tf tree," "the graph" | ~$122k–$194k, top ~$239k ([ZipRecruiter](https://www.ziprecruiter.com/Salaries/Robotics-Software-Engineer-Salary)) | In-house core; the role you build a team around |
| **Integration engineer / systems integrator (SI)** | Make a new robot/sensor/actuator reliable, controllable and deployment-ready inside an existing stack; wire CAN/EtherCAT/serial; write the drivers | Device drivers, HAL, SDK integrations, bring-up checklists | C++/Python on Linux, ROS 2, CAN/EtherCAT/RS-485/I²C/SPI ([Glassdoor](https://www.glassdoor.com/Job/integration-engineer-robotics-jobs-SRCH_KO0,29.htm)) | "bring-up," "driver," "HAL," "the bus," "it enumerates," "deployment-ready" | ~$84k–$172k ([ZipRecruiter](https://www.ziprecruiter.com/Jobs/Robotics-Integrator)) | Mixed — in-house for product OEMs, contracted as SI firms for factory floors |
| **Controls engineer** | Design the feedback laws that make the hardware move correctly and stably — trajectory tracking, balance, force control | Controller designs, gain sets, state-space models, stability proofs | MATLAB/Simulink, C++ (real-time), state-space, LQR, PID, model identification ([WPILib](https://docs.wpilib.org/en/stable/docs/software/advanced-controls/state-space/state-space-intro.html)) | "plant," "gains," "poles/zeros," "settling time," "state estimate," "the loop closes" | ~$83k–$156k+ ([ZipRecruiter](https://www.ziprecruiter.com/Jobs/Full-Time-Robotics-Control-Engineer)); PhDs at the top | In-house — too close to the IP to contract |
| **Behaviour / autonomy engineer** | Encode mission logic — what the robot *decides to do* and in what order; choose and tune the behaviour-tree / state-machine architecture | Behaviour trees, state machines, mission definitions, autonomy policies | C++/Python, `BehaviorTree.CPP`, Nav2, FSM libraries, ROS 2 ([Robohub](https://robohub.org/introduction-to-behavior-trees/)) | "tick," "node status (`SUCCESS`/`FAILURE`/`RUNNING`)," "fallback/sequence," "blackboard," "the tree" | overlaps software-engineer band, often $130k+ | In-house core |
| **Perception / ML engineer** | Make the robot see and understand — detection, segmentation, tracking, pose estimation, sensor fusion | Trained models, perception pipelines, annotation/training infrastructure | Python, PyTorch/TensorFlow, OpenCV, ROS 2, LiDAR/camera/IMU fusion ([ZipRecruiter](https://www.ziprecruiter.com/Jobs/Robotics-Perception-Engineer)) | "detection," "mAP," "point cloud," "the model," "ground truth," "the dataset" | ~$97k–$300k ([ZipRecruiter](https://www.ziprecruiter.com/Jobs/Robotics-Perception-Engineer)) | In-house; data annotation often contracted out |
| **HRI designer (human-robot interaction)** | Design how the robot *behaves around people* — signalling intent, building trust, safe proximity, social legibility | Interaction models, behaviour specs for social cues, HRI study reports | Figure/Sketch, study protocols, sometimes Unity prototypes; research methods | "affordance," "legibility," "intent signalling," "trust calibration," "mental model" ([IxDF](https://ixdf.org/literature/topics/human-robot-interaction)) | research-leaning; $110k–$180k where staffed | Rare in-house; often academic-adjacent or contracted |
| **Robotics UX designer** | Design the *screens* humans use to command, monitor and intervene — operator consoles, fleet dashboards, teach pendants | Figma files, design systems, operator-flow specs, prototypes | Figma, design tokens, prototyping tools; sometimes React for hi-fi prototypes | "flow," "operator," "affordance," "information hierarchy," "the happy path," "edge case" | general product-UX bands; the *robotics* specialisation is scarce | Often contracted (agencies) — see §3 |
| **Frontend / operator-UI engineer** | Build the actual operator web app — telemetry views, teleop controls, video, the fleet table | Operator web application, component library | TypeScript, React/Next.js, WebSocket, WebRTC, roslibjs / Foxglove SDK | "the dashboard," "the socket," "latency," "the panel," "component" | web-engineer bands; robotics-aware is a premium | Mixed; the studio's natural seat |
| **RobOps / fleet operations engineer** | Keep deployed fleets alive — monitoring, escalation, OTA updates, field repair, incident response | Runbooks, dashboards, incident reports, OTA release notes | Fleet platforms (Formant/InOrbit), SSH, monitoring stacks, ticketing ([InOrbit](https://www.inorbit.ai/robops)) | "fleet," "uptime," "escalation," "the alert," "on-call," "incident," "rollback" | field+software hybrid; ~$90k–$150k | In-house ops; some firms (Robotic Crew) offer nearshore RobOps talent |
| **Simulation engineer** | Build and maintain the virtual environments where everything is tested before hardware — and the sim-to-real bridge | Simulation worlds, sensor models, sim-to-real test harnesses | Gazebo, NVIDIA Isaac Sim, MuJoCo, USD, ROS 2 bridge ([NVIDIA](https://developer.nvidia.com/isaac/sim)) | "the sim," "domain randomisation," "sim-to-real gap," "the world file," "headless run" | $90k–$240k range at NVIDIA-tier firms ([ZipRecruiter](https://www.ziprecruiter.com/Jobs/Nvidia-Robotics)) | In-house at scale; contracted for one-off environments |
| **Functional-safety / certification engineer** | Prove the system is safe — hazard analysis, PL/SIL ratings, conformance to ISO 10218 / ISO 13849 / ISO/TS 15066 / ANSI R15.08 | Hazard logs, safety cases, conformance dossiers, the PL/SIL rating | Safety-analysis tools, requirements tools, the standards themselves ([ISO 10218-1:2025](https://www.iso.org/standard/73933.html)) | "hazard," "PLd/PLe," "the safety case," "residual risk," "conformance," "the standard" | median ~$98k, range $83k–$191k ([ZipRecruiter](https://www.ziprecruiter.com/Jobs/Robotic-Safety-Engineer)) | Often contracted — a notarial, episodic role; certification bodies (UL, TÜV) provide it |
| **Robotics product manager** | Own the *why* and the *roadmap* — decide which capabilities ship, in what order, for which customer | PRDs, roadmaps, requirements, release plans | Linear/Jira, roadmap tools, docs; reads everyone else's artefacts | "requirement," "roadmap," "epic," "the customer," "MVP," "priority," "the spec (loosely)" | base $118k–$287k depending on tier ([Built In](https://builtin.com/jobs/product/robotics)) | In-house |

### 1.1 Reading the catalogue

Three patterns matter for the methodology question:

**The role count collapses under headcount, the artefact count does not.** A 12-person robotics startup might have *four* humans covering all twelve rows — one "ROS person" doing software + behaviour + integration, one controls PhD, one perception person, and a generalist doing sim + a bit of frontend, with UX, HRI, safety and PM bought in as needed. But that generalist still produces a behaviour tree *and* a launch file *and* a Gazebo world, and still mentally context-switches between three vocabularies. The Babel problem (§5) is not primarily *interpersonal*; it is *intrapersonal too*. One person juggling four file formats is the same coordination tax as four people, minus the meetings.

**Scarcity predicts the buy/build line.** Controls and perception sit at the top of the salary bands and are never contracted — they are the IP. Safety and HRI sit lower-frequency and *are* contracted, episodically. UX is contracted because the *robotics* specialisation is too thin to justify a permanent hire below a certain company size. **This tells the studio exactly where its wedge is**: nobody contracts out their behaviour tree, but plenty of OEMs will contract out the operator surface and the safety dossier surface. The methodology should not try to own the controls engineer's loop; it should own the seams the contracted roles already cross.

**The PM is the only role that already reads everyone's artefacts** — and reads them badly, because there is no common format. The PM's "the spec" is a loose Confluence/Linear prose doc that the controls engineer ignores and the safety engineer treats as non-binding. A methodology that gave the PM a *real* artefact — one the other roles are obliged to read — is, structurally, the most plausible adoption path. The PM is the natural sponsor.

---

## 2. Who writes adapters specifically

The studio's Gateway thesis (`robot-operator-interfaces.md` §3) lives or dies on the *adapter* layer — the driver/bridge/HAL between a robot and the rest of the world. So: who builds that layer today, and is there a buyer who self-identifies as needing it?

### 2.1 "Adapter writer" is not a job title

There is no "Adapter Engineer" on LinkedIn. The work is **diffuse across at least four overlapping titles**:

- **Integration engineer** — the closest fit. Job postings explicitly say "writing and maintaining device drivers and sensor interfaces for real robot hardware," "owning the communication layer between onboard compute and hardware subsystems," "designing and implementing hardware abstraction layers" ([Glassdoor integration-engineer postings](https://www.glassdoor.com/Job/integration-engineer-robotics-jobs-SRCH_KO0,29.htm)). This *is* adapter writing, but the title frames it as "integration," a verb about *fitting things together*, not about *the adapter as an object*.
- **"Software Engineer, Robotics Integrations"** — Tesla and Field AI post this exact title ([Tesla careers](https://www.tesla.com/careers/search/job/software-engineer-robotics-integrations-250588)). Closer still — "Integrations" plural is almost "adapters" — but it is an OEM-internal title, not an industry-wide one.
- **Embedded software engineer** — owns the bus-level and firmware-adjacent half of the adapter (CAN, EtherCAT, SPI). On many teams the embedded engineer writes the *bottom* of the driver and the ROS developer writes the *top* (the node that publishes the topic). The adapter is literally split across two people with two titles.
- **Robotics deployment / support engineer** — writes the *last-mile* adapters: the connector to the customer's WMS, the bridge to their MES, the glue to their existing fleet manager. This shades into RobOps.

The phrase **"robot enablement"** appears in vendor marketing (and informally inside large OEMs as a team name) but is not a recognised job title. It usually denotes an internal team whose job is "make new hardware usable by the rest of the company" — i.e. an adapter team by another name.

**Why the diffuseness is the finding.** The adapter layer is real, technically demanding, and universally needed — *and it has no professional identity*. There is no "adapter guild," no conference track, no canonical certification, no Stack Overflow tag. For the studio this cuts both ways:

- *Opportunity:* there is no incumbent profession defending its turf. A methodology that says "here is how you spec an adapter" is not stepping on anyone's established practice.
- *Risk:* there is **no buyer who wakes up thinking "I need to buy adapter tooling."** The studio cannot sell to "adapter writers" because they do not know they are adapter writers. The sale has to be framed as something the buyer *does* identify with — "integration," "bring-up time," "deployment readiness," "operator surface."

### 2.2 The systems-integrator (SI) firms — adapters as a service

When an OEM or a factory does not write its own adapters, it hires an SI. SIs are the *contracted* face of adapter work. The landscape splits into three tiers:

| Tier | Firms (examples) | What they do | Relevance to adapter layer |
|---|---|---|---|
| **Industrial-arm SIs (the classic floor integrators)** | [JR Automation](https://www.jrautomation.com/solutions/technology-integrations/robotics-integration) (Hitachi-owned), [ATS Corporation](https://www.atsautomation.com/), Comau, [Jabil](https://us.metoree.com/categories/102019/), Optimation, RNA Automation | Design, build and commission robotic work cells — arm + fixturing + vision + PLC — for manufacturing customers | They write the *PLC/OPC-UA/fieldbus* adapters between the robot and the factory. ROS-light, automation-heavy. Hundreds of them worldwide; consolidating (Hitachi bought JR Automation; many regional shops) |
| **AMR / mobile-robot integrators** | InOrbit's partner network, WAKU Robotics, regional logistics integrators | Deploy and interconnect mobile-robot fleets in warehouses; make multi-vendor fleets cooperate | They live on VDA 5050 / MassRobotics adapters — exactly the studio's Gateway territory ([InOrbit ecosystem](https://www.inorbit.ai/)) |
| **Robotics dev / nearshore talent shops** | Robotic Crew (nearshore robotics dev + ops talent — [The Robot Report](https://www.therobotreport.com/robotic-crew-offers-nearshore-robotics-development-operations-talent/)), boutique ROS consultancies, RSI | Supply ROS 2 engineers and integration capacity on contract; write drivers, bring up hardware, build the glue | The closest analogue to what a Moldova-based studio *is* — and therefore both a model and a competitor |

The "top robotic systems integrator" rankings (Jabil, Optimation, RNA Automation per [Metoree](https://us.metoree.com/categories/102019/)) are dominated by **industrial-automation** firms whose "robot" is an arm bolted to a floor. That is a different world from the ROS-2/mobile-robot world the studio's operator-surface thesis targets. The mobile-robot and humanoid SIs — the ones who live on VDA 5050, ROS 2 and MassRobotics — are a *much* smaller and younger set, and they are the studio's actual peers and channel.

**Implication.** The classic SIs are not competitors and not channel — they are too automation-PLC-shaped. The AMR integrators and the nearshore ROS shops are *both* a channel (they could resell a studio Gateway) *and* a competitor (they could hand-roll the adapter themselves and bill the hours). The studio's edge over a nearshore body shop is a *reusable substrate*: the body shop re-writes the VDA 5050 adapter every engagement; the studio writes it once and re-themes it. That is the same buy/build argument as everywhere else in the studio's thesis.

---

## 3. Who does robot UI/UX specifically

### 3.1 The discipline exists — and it is small

Robotics UX is a recognised specialisation, with two halves that are routinely confused:

- **HRI design** — how the *robot itself* behaves around people: motion legibility, intent signalling, proximity, sound, light, social cues. This is closer to product/industrial design and behavioural research than to screen design. Practitioners cite "affordance," "legibility," "trust calibration," "mental model" ([IxDF HRI](https://ixdf.org/literature/topics/human-robot-interaction)). It typically demands a CS/HCI/cognitive-science degree, often a master's or PhD ([Gladeo](https://losangeles.gladeo.org/career/human-robot-interaction-designer)).
- **Operator/screen UX** — the *software surfaces* humans use to command, monitor and intervene: teach pendants, operator consoles, fleet dashboards, teleop UIs. This is product UX with a robotics domain overlay. **This is the studio's lane.**

Evidence the discipline is real: Amazon staffs "Senior UX Designer, Amazon Industrial Robotics" and "Principal UX Design Researcher, Amazon Industrial Robotics" as distinct, senior roles ([Amazon jobs](https://www.amazon.jobs/en/jobs/3172158/senior-ux-designer-amazon-industrial-robotics)); Boston Dynamics recruits "on-screen and physical UX/UI Designers specialized in Robotics" alongside 3D animators and HRI experts; surgical-robotics firms (Intuitive-class) staff dedicated robotics-UX teams because regulatory human-factors files demand it.

Evidence it is **small**: the search surface for "robot UX designer" is thin and dominated by a handful of large employers. There is no robotics-UX conference of consequence, no robotics-UX design system anyone cites, no robotics-UX equivalent of "Material Design." Compared to general product UX it is a rounding error. The scarcity is exactly *why* it gets contracted out below a certain company size — which is the studio's opening.

### 3.2 The agencies and studios doing robotics UX

There is no large, pure-play "robotics UX agency." Instead there are: (a) generalist product-design agencies with a robotics *portfolio line*, (b) engineering firms with an HRI *practice*, and (c) in-house teams at the big robot companies.

| Studio / firm | Type | Robotics-UX evidence | Notes |
|---|---|---|---|
| **[argodesign](https://argodesign.com/)** | Generalist product/industrial design consultancy (Austin; founded 2014 by ex-Frog CCO Mark Rolston; sold to a $22B IT company — [Fast Company](https://www.fastcompany.com/90239194/argodesign-sells-to-a-22-billion-it-company)) | Designed the **physical form of the Apptronik Apollo humanoid** for the NASA-backed program ([SiliconHills](https://www.siliconhillsnews.com/2024/01/13/from-frog-design-to-argodesign-mark-rolstons-journey-of-innovation-creativity-and-ai-insights/)) | The single most visible robotics-design credential in the agency world — but it is *industrial design*, the robot's body, not the operator screen |
| **[IIIMPACT](https://www.iiimpact.io/)** | Boutique enterprise product-UX + dev agency (Austin; ~17 years; Inc. 5000) | Explicit robotics practice; publishes "Key Principles of UX Design for Robotic Applications" and "Effective UX Testing Methods for Robotic Interfaces" ([IIIMPACT robotics](https://www.iiimpact.io/news/key-principles-of-ux-design-for-robotic-applications)) | Closest to a *screen-UX-for-robots* agency that markets itself as such — and therefore the closest direct comparator to the studio's UX offering |
| **[Punchcut](https://punchcut.com/)** | Design + innovation consultancy (San Francisco) | Markets "designing for technologies that don't yet have established patterns: AI conversational interfaces, spatial computing, multimodal systems" — adjacent to robot UX rather than explicitly robotics | Strong on AI/multimodal; would compete on the AI-companion narrative angle |
| **[Boston Engineering](https://www.boston-engineering.com/solutions/technical-innovation/robotics/robotics-design-and-application-expertise/human-robot-interaction-design/)** | Engineering services firm with a robotics + HRI practice | Sells "Human-Robot Interaction Design" as a named service, alongside robotic system integration | Engineering-led, not design-led — HRI as a sub-practice of a robotics engineering shop |
| **In-house teams** | Amazon Industrial Robotics, Boston Dynamics, Intuitive Surgical, surgical-robotics firms generally | Dedicated staff UX + HRI roles; regulatory human-factors files force it in medical/surgical | Large OEMs internalise it; everyone below ~Series B contracts it |
| **Frog, IDEO, and other generalist giants** | Generalist design giants | Occasional robotics projects within a broad portfolio; no robotics *line of business* | Robotics is a project type, not a practice |

**The honest read.** Robotics UX-as-a-discipline is (a) genuinely real, (b) genuinely small, (c) *bifurcated* between industrial design (the robot's body — argodesign's Apollo) and screen UX (the operator surface — IIIMPACT's lane), and (d) **mostly contracted below Series B**, because no robot startup with 15 engineers hires a permanent robotics-UX specialist. That last point is the entire commercial case for the studio's UX offering: the customers exist, the work is real, the specialisation is too thin to hire for full-time, and there is no dominant agency holding the category. IIIMPACT is the proof the slot is monetisable and the proof it is not yet *owned*.

---

## 4. The biggest and best UI/UX + observability vendors for robotics

This section ranks the operator/observability vendors. It overlaps deliberately with `robot-operator-interfaces.md` §2 but reframes them as a *supplier landscape* — who is biggest, who is best, and why those diverge — rather than as competitors to a specific studio product.

### 4.1 The vendor comparison table

| Vendor | What it is | Funding / scale | Customers | Sells: product vs service | Pricing model (where public) |
|---|---|---|---|---|---|
| **[Foxglove](https://foxglove.dev/)** | Multimodal data + visualization platform for "Physical AI" — viz, MCAP recording format, data lake | **$40M Series B**, Bessemer-led, Nov 2025; ~$58M+ total since 2021 ([BusinessWire](https://www.businesswire.com/news/home/20251112126106/en/Foxglove-Raises-$40-Million-Series-B-to-Power-the-Future-of-Physical-AI)) | "Hundreds of customers," **tens of thousands of developers**; NVIDIA, Amazon, Anduril, Wayve, Dexterity, Shield AI ([The Robot Report](https://www.therobotreport.com/foxglove-raises-40m-scale-data-platform-roboticists/)) | **Product**, self-serve | Free tier; team plans on request; enterprise contract |
| **[Formant](https://formant.io/)** | Cloud data + operations platform — observability + teleop + analytics for fleets | **~$45M total over 3 rounds**; Series B Oct 2023, $21M, led by BMW i Ventures ([Tracxn](https://tracxn.com/d/companies/formant/__57eZfBvFCaCQUMRBcO3pdriFRxYi-3vmPke--QFN7h0)) | Cobalt Robotics, Built Robotics, Bear Robotics; pilot-stage humanoid + AMR firms | **Product** with a services/onboarding wrap | Free tier for individuals; paid tiers undisclosed publicly |
| **[InOrbit.AI](https://www.inorbit.ai/)** | RobOps platform — multi-vendor fleet orchestration, missions, RobOps Copilot (NL queries over robot data) | **$10M Series A**, Sept 2025, co-led by L'ATTITUDE Ventures + Globant Ventures ([BusinessWire](https://www.businesswire.com/news/home/20250930673347/en/InOrbit.AI-Secures-Series-A-Funding-to-Scale-Robot-Orchestration-Platform-Unlocking-a-New-Era-of-Software-Driven-Physical-Workflows)) | Colgate-Palmolive, Genentech, Google, Qualcomm; "a dozen+ robot manufacturers"; MiR Go partner, OTTO via VDA 5050 | **Product**, sales-led; Developer Edition for self-serve | Enterprise; quoted ranges $50k–$500k/yr by fleet size |
| **[Boston Dynamics — Orbit](https://bostondynamics.com/products/orbit/)** | Cloud fleet management for Boston Dynamics robots — centralized dashboards for Spot, Stretch, eventually Atlas | Backed by Hyundai (BD ownership); not separately funded | The Boston Dynamics installed base — large enterprise (Spot Explorer kit $74.5k; full enterprise system $150k–$195k/robot — [easy-robots](https://www.easy-robots.com/en/articles/boston-dynamics-spot-review)) | **Product**, bundled with hardware | Sold with the robot / enterprise contract; cloud, NA-first |
| **[Transitive Robotics](https://transitiverobotics.com/)** | Open-source robotics framework + à-la-carte web "capabilities" (teleop, video, etc.) | Small / early; open-source-led ([Crunchbase](https://www.crunchbase.com/organization/transitive-robotics)) | Developer-led adoption; InOrbit ecosystem partner | **Product**, à-la-carte; open-source core | **Per-capability, per-robot/month** — e.g. **$15/robot/mo** for live video; some capabilities free ([The Robot Report](https://www.therobotreport.com/transitive-robotics-opens-beta-of-remote-operations-solution/)) |
| **[Cogniteam — Nimbus](https://www.cogniteam.com/)** | Low-code AIoT + robotics cloud platform — robot dev, deployment, monitoring | **$5.6M Series A** for Nimbus ([Financial IT](https://financialit.net/news/artificial-intelligence/cogniteam-raises-56-million-series-enhance-nimbus-low-code-ai-robotics)) | NVIDIA, AAEON, Advantech partnerships; developer-led | **Product**, low-code | Cloud subscription; tiers not fully public |
| **[Freedom Robotics](https://www.freedomrobotics.com/)** | Fleet management + dev tools — build, operate, support, scale robot fleets | Seed-stage; **$6.6M round Sept 2025**; **independent**, not acquired ([PitchBook](https://pitchbook.com/profiles/company/277361-11)) | Industrial / automotive / logistics fleet operators | **Product** | Subscription; not fully public |
| **[Cyberwave](https://cyberwave.com/)** | "AI-defined robotics platform" — one API for any robot/drone/sensor; programmable digital twins; teleop for human-in-the-loop | **€7M**, Oct 2025, led by United Ventures (Milan; founded 2025) ([EU-Startups](https://www.eu-startups.com/2025/10/milan-based-cyberwave-secures-e7-million-to-simplify-ai-driven-automation/)) | Pilots with European automotive / construction / logistics manufacturers | **Product** + two-sided marketplace (hardware makers integrate once) | Early; not public |
| **[Rocos](https://www.crunchbase.com/organization/rocos)** | *(Historical)* robotic cloud / fleet platform | **Acquired by DroneDeploy, Aug 2021** ([Robotics 24/7](https://www.robotics247.com/article/dronedeploy_acquires_rocos_automate_ground_air_jobsite_data_collection)) | — | Folded into DroneDeploy's jobsite-data product | n/a |

### 4.2 Who is "biggest"

There are three different "biggest," depending on the axis:

- **Biggest by capital and developer reach in the pure-play robotics-data category: Foxglove.** $40M Series B from Bessemer, ~$58M+ total, tens of thousands of developers, and the de facto recording format (MCAP) is theirs. In the specific niche of "robotics visualization / data tooling vendor," Foxglove is the clear leader and the others are not close on capital.
- **Biggest by enterprise installed base inside a single ecosystem: Boston Dynamics Orbit.** Orbit ships with a hardware franchise where a single robot costs $75k–$195k. The *number of dashboards* is small, but the *revenue and enterprise weight per seat* dwarfs Foxglove's developer base. The catch: Orbit is single-vendor by design — it manages Boston Dynamics robots only.
- **Biggest by breadth of operations footprint (fleets actually run in production through the tool): InOrbit and Formant.** InOrbit's Colgate-Palmolive / Genentech / Google logos and "a dozen+ robot manufacturers" represent real multi-vendor fleets in production. Formant's BMW-i-Ventures backing and Cobalt/Built/Bear customers likewise. Neither has Foxglove's capital, but both run more *operations* than Foxglove does — Foxglove is consciously a *data* company, not an *operations* company.

So the honest answer to "who is the largest provider of UI/UX/operator tooling for robots in the world": **Foxglove for the pure-play robotics-tooling category by capital and reach; Boston Dynamics Orbit if you count single-vendor enterprise fleet software; and the truly *largest* operator-facing software footprint in absolute terms is none of these — it is the in-house FetchCore / LocusONE / MiR Fleet / FlightHub-2 consoles that ship bundled inside the big AMR and drone OEMs (`robot-operator-interfaces.md` §1).** The pure-play vendors are large in *mind-share*; the bundled OEM consoles are large in *deployed seats*.

### 4.3 Who is "best" — and why it diverges from biggest

"Best" depends entirely on the buyer's job-to-be-done:

- **Best for a roboticist who needs to debug and visualize data: Foxglove.** Nothing matches its panel system, MCAP, and the high-performance `foxglove_bridge`. If the job is "see what the robot's sensors are doing," Foxglove wins.
- **Best for a fleet operator who needs teleop + monitoring + analytics in one product: Formant.** Out-of-the-box teleop UI, fleet onboarding, Formant Analytics, Slack/PagerDuty/Jira integrations. Foxglove makes you assemble the operations layer yourself; Formant ships it.
- **Best for a heterogeneous multi-vendor fleet: InOrbit.** It is the only one in the list architected around *cross-vendor* orchestration (VDA 5050, MassRobotics, Open-RMF) as a first-class concern, and its RobOps Copilot is the closest any incumbent gets to a natural-language narrative layer over robot data ([InOrbit RobOps](https://www.inorbit.ai/robops)).
- **Best price/ergonomics for a tiny team: Transitive Robotics.** À-la-carte, $15/robot/month for video, open-source core — the lowest-friction way to put a remote-teleop capability on a robot without an enterprise contract.

**Biggest ≠ best because the category is not one category.** Foxglove won the *data/visualization* race; it deliberately did not enter the *operations* race. Formant and InOrbit are "better" for an operator precisely because Foxglove chose not to compete there. And *all* of them are "worse" than a bundled OEM console for the OEM's own customers, because none of them white-label — the OEM cannot ship "MyBrand Console" with any of these inside. That structural gap (branded surface, vertical UX, AI narrative) is the studio's documented opening (`robot-operator-interfaces.md` §5) and it exists *because* the incumbents optimised for being the brand, not for disappearing behind one.

A note on the **AI-narrative frontier**: InOrbit's RobOps Copilot is the first incumbent move into "ask questions of your robot data in natural language." It is a real signal that the narrative layer is coming. It is also, today, a query tool bolted onto an enterprise platform — not a designed operator-companion experience. The window for a *taste-led* companion layer is open but visibly closing.

---

## 5. The Tower of Babel problem

This is the section the unified-methodology thesis actually rests on. The claim under test: the roles in §1 are not just *different* — they are *mutually unintelligible*, each working in a vocabulary, a toolchain, a file format and a definition of "done" that the others cannot read. If that is true, a shared spec layer has a job. If it is false — if they already share enough — the methodology is solving a non-problem.

It is true. Here is the evidence laid out side by side.

### 5.1 The twelve vocabularies

The same robot, the same mission, described by each role in their native terms — and notice that *no two descriptions share a noun*:

| Role | How they describe "the robot picks up the box and carries it to the dock" |
|---|---|
| **Robotics software engineer** | "The `pick_box` action server publishes on `/manipulation/goal`; the nav stack subscribes to `/cmd_vel`; tf has to resolve `base_link → dock_frame`." |
| **Integration engineer** | "The gripper driver enumerates on the EtherCAT bus, the arm HAL exposes a `grasp()` call, the base controller takes the velocity command over the CAN adapter." |
| **Controls engineer** | "The arm tracks the grasp trajectory with the impedance controller; the base closes the velocity loop; settling time under 200 ms, no overshoot past 2%." |
| **Behaviour / autonomy engineer** | "The mission tree has a `Sequence`: tick `DetectBox` → `ApproachBox` → `Grasp` → `NavigateToDock` → `Release`; if `Grasp` returns `FAILURE`, the `Fallback` retries twice then aborts." |
| **Perception / ML engineer** | "The detector runs at 30 Hz, box class confidence above 0.85, pose estimate from the point cloud with covariance under threshold." |
| **HRI designer** | "The robot signals intent before moving — a light cue and a slow lead-in motion — so a nearby worker forms a correct mental model of where it is going." |
| **Robotics UX designer** | "The operator sees the mission as five steps in a progress rail; if the grasp fails, an exception card surfaces with the last camera frame and a 'retry / take over' choice." |
| **Frontend / operator-UI engineer** | "The mission-state WebSocket pushes `step` events; the progress component re-renders; the video panel switches to the wrist camera on the `grasp` event." |
| **RobOps engineer** | "If the mission aborts twice in an hour it pages on-call; the runbook says check gripper calibration; the OTA channel has the fix in `v2.4.1`." |
| **Simulation engineer** | "The pick-and-carry scenario runs headless in Isaac Sim with domain randomisation on box pose and lighting; the sim-to-real gap on grasp success is 6%." |
| **Functional-safety engineer** | "The carry phase is hazard H-07 (load drop); mitigation is speed-and-separation monitoring per ISO/TS 15066; residual risk acceptable at PLd." |
| **Product manager** | "Pick-and-carry is the Q3 milestone; the requirement is 98% completion at the customer's dock; it's the top roadmap item for the logistics customer." |

Twelve descriptions of one event. The overlap in nouns is essentially zero. "The robot picks up the box" is not one fact in this organisation — it is twelve facts, in twelve namespaces, that happen to be correlated.

### 5.2 The divergent toolchains, formats, reviews and "done"

| Role | Primary tool | Native file format | Review process | Definition of "done" |
|---|---|---|---|---|
| **Software engineer** | VS Code, ROS 2 CLI | `.cpp`/`.py`, `.launch.py`, `package.xml` | GitHub PR, code review | Nodes run, topics flow, CI green |
| **Integration engineer** | Oscilloscope, bus analyser, VS Code | driver source, bring-up checklist (often a wiki page) | informal — "it enumerated, it moved" | Hardware is controllable and deployment-ready |
| **Controls engineer** | MATLAB/Simulink, plotting tools | `.slx`, `.mat`, gain tables | plot review — "look at the step response" | Step response meets spec; stability margin proven |
| **Behaviour engineer** | Behaviour-tree editor (Groot), VS Code | `.xml` behaviour tree, FSM source | tree walkthrough in sim | Tree ticks `SUCCESS` across scenarios |
| **Perception engineer** | Jupyter, PyTorch, annotation tools | `.pt`/`.onnx` weights, datasets, training configs | metric review — "mAP went up" | Model hits the target metric on the eval set |
| **HRI designer** | Figma, study protocols | study reports, interaction specs (slides/docs) | study readout, design crit | Users form the intended mental model in testing |
| **UX designer** | Figma | `.fig`, design-system files, flow specs | design crit | Users complete the flow unaided in usability testing |
| **Frontend engineer** | VS Code | `.tsx`, component library | GitHub PR | UI works against the live socket, CI green |
| **RobOps engineer** | Fleet platform, ticketing | runbooks, incident reports, dashboards | post-incident review | Fleet uptime SLO met; incident closed |
| **Simulation engineer** | Isaac Sim / Gazebo | `.usd`/`.sdf` world files, scenario configs | sim run review | Scenario passes; sim-to-real gap quantified |
| **Safety engineer** | Safety-analysis + requirements tools | hazard log, safety case dossier | formal review with a certification body | Hazard log closed, PL/SIL rating signed off |
| **Product manager** | Linear/Jira, Confluence, Notion | PRD prose, roadmap | stakeholder sign-off | Milestone shipped, customer accepts |

Four facts jump out:

1. **No two roles share a file format.** Not one. `.slx` ≠ `.xml` behaviour tree ≠ `.fig` ≠ `.pt` ≠ hazard-log spreadsheet ≠ `.usd` ≠ PRD prose. There is no artefact a controls engineer and a UX designer both open.
2. **No two roles share a review ritual.** A code PR, a plot review, a design crit, a sim run, a formal certification-body review, a post-incident review — six incompatible review cultures. A behaviour engineer cannot meaningfully review a hazard log; a safety engineer cannot meaningfully review a design crit.
3. **"Done" is twelve different events.** This is the deepest divergence. The behaviour engineer's "done" (`SUCCESS` in sim) happens *months* before the safety engineer's "done" (signed PL rating). The PM's "done" (customer accepts) is downstream of all eleven others. When a PM says "are we done?" the honest answer is "for whom?" — and there is no artefact that answers it.
4. **The PM's artefact is prose, and prose is non-binding.** The PM's PRD is the *only* artefact that even *attempts* to span roles — and it is unstructured prose that the controls engineer treats as a suggestion and the safety engineer treats as informational. The one cross-role artefact that exists is the one with no enforcement.

### 5.3 The seam where the cost lands

The Babel tax is not paid evenly. It concentrates at **handoffs**:

- **Behaviour engineer → UX designer.** The behaviour tree has a node called `Grasp` that can return `FAILURE`. The UX designer needs to design the "grasp failed" screen. Today this handoff is a meeting and a Slack thread, because the UX designer cannot read the `.xml` tree and the behaviour engineer does not think in screens. *The failure modes of the tree and the exception states of the UI are the same set — represented twice, by two people, in two formats, kept in sync by hope.*
- **Integration engineer → operator-UI engineer.** The adapter exposes some set of telemetry fields. The operator UI displays some set of telemetry fields. These should be the same set. They drift, because the adapter's "interface" lives in driver source and the UI's "interface" lives in TypeScript types, and nothing reconciles them.
- **PM → everyone.** The PM writes "98% completion." The behaviour engineer, the controls engineer, the perception engineer and the safety engineer each independently decide what "completion" means for their layer. They are not wrong; they are unaligned, because the requirement was prose.
- **Everyone → safety engineer.** The safety engineer has to reconstruct, from eleven other people's artefacts, a hazard analysis. They are doing *archaeology* on a system that was never documented in a form they can read.

**This is the precise problem a shared spec layer addresses** — not by replacing the twelve tools, but by giving the *handoffs* a common artefact. The behaviour engineer keeps Groot; the UX designer keeps Figma; but the *list of failure modes* lives once, in a spec both of them read.

---

## 6. Where they could converge

The honest section. The Babel diagnosis in §5 is real, but the graveyard of attempts to fix it is also real. Anyone proposing a unified methodology has to explain why MBSE, ROS REPs, the interoperability standards and digital twins each got part way and stalled — and then say, without overclaiming, where a lightweight markdown-spec methodology could actually sit.

### 6.1 What has been tried, and why each stalled

**MBSE / SysML — the most ambitious attempt, the clearest cautionary tale.** Model-Based Systems Engineering with SysML is *exactly* the "one model all roles share" thesis, attempted seriously for two decades. It is used in aerospace and defence robotics. And the post-mortem literature is brutal and consistent:

- SysML "is hard to understand and cumbersome to use," is "not fully mature," and is "interpreted in slightly different ways by many software vendors" ([SpecInnovations](https://specinnovations.com/blog/what-are-the-challenges-of-adopting-mbse-tools)).
- Adoption is "a change process affecting a very complex system where the most important system elements are humans" — i.e. the blocker is organisational, not technical ([one-sys.eu](https://www.one-sys.eu/en/post/from-model-to-mindset-the-real-challenge-of-model-based-systems-engineering)).
- "There is no universally recognized methodology that explains in a practical way how to apply MBSE to real projects" ([INCOSE / Wiley](https://incose.onlinelibrary.wiley.com/doi/full/10.1002/sys.21717)).
- ROS and MBSE "often lack integration when used together" — the modelling world and the actual-robot-software world do not connect ([ResearchGate, MBSE for robotics](https://www.researchgate.net/publication/387663587)).

**Why it stalled:** SysML demanded that every role learn a heavyweight formal modelling language and an expensive tool, *in addition to* their own toolchain — and gave them, in return, diagrams. The controls engineer was asked to abandon Simulink-as-truth for a SysML block diagram and saw no benefit. The cost was front-loaded and per-role; the benefit was diffuse and accrued to the systems engineer. **The lesson is decisive: a unification artefact that asks each role to abandon its native tool will be rejected by each role.**

**ROS REPs (ROS Enhancement Proposals).** REPs successfully standardised *technical conventions* within the ROS world — coordinate frames (REP-103), units, package layout. They are a genuine convergence success. But their scope is narrow by design: they unify *ROS developers with each other*. They say nothing to the controls engineer, the UX designer, the safety engineer or the PM. REPs prove convergence is possible — *within a single role*. They do not attempt cross-role.

**VDA 5050 / MassRobotics AMR Interoperability Standard.** These standardise *robot-to-robot and robot-to-fleet-manager* communication so multi-vendor AMR fleets cooperate ([MassRobotics](https://www.massrobotics.org/what-is-the-massrobotics-amr-interoperability-standard/)). VDA 5050 is a real, adopted success in EU logistics. But — same shape as REPs — it unifies *one seam*: the wire protocol between a robot and an orchestrator. It says nothing about how the behaviour engineer and the UX designer agree on failure modes. And even within its narrow scope, adoption is partial: "about 30 organizations participating, about a dozen actively involved" in MassRobotics. Standards converge a seam; they do not converge a methodology.

**Digital-twin platforms (Isaac Sim / Omniverse Mega, MeROS metamodels).** These promise a shared *simulated reality* all roles point at. They are powerful and the simulation engineer loves them. But they converge the *test environment*, not the *intent*. A digital twin tells you what the system *does*; it does not capture what the PM *wanted*, why the safety engineer flagged a hazard, or what mental model the HRI designer intended. And they are heavyweight — another expensive specialist tool, not a shared lightweight artefact.

**The pattern across all four:** every prior attempt either (a) converged *one seam* well and left the other eleven (REPs, VDA 5050), or (b) attempted full convergence with a heavyweight artefact and was rejected role-by-role on cost (MBSE/SysML), or (c) converged the test environment rather than the intent (digital twins). Nobody has converged *intent* with a *lightweight* artefact.

### 6.2 Where a lightweight spec-driven methodology could realistically sit

Given that history, the studio's OpenSpec-style methodology — markdown `proposal.md` / `design.md` / `tasks.md` plus capability specs plus sequence diagrams — should make a **deliberately modest** claim. Not "the universal model." The realistic claim is three-layered:

**1. The shared *review surface*, not the shared *tool*.** The methodology's job is not to replace Simulink, Groot, Figma, PyTorch or the hazard log. It is to be the one artefact *all twelve roles read*, even though each still produces their work in their own tool. The behaviour engineer keeps Groot but the *list of mission steps and failure modes* is also written, in prose-plus-diagram, in `design.md` — where the UX designer, the safety engineer and the PM can read it. The spec is the *index* and the *contract*; the native tools remain the *workshops*. This is the seam-with-a-common-artefact model from §5.3, generalised.

**2. The handoff contract, especially behaviour ↔ UX and integration ↔ frontend.** The two seams where the studio actually operates — behaviour-tree failure modes ↔ operator exception screens, and adapter telemetry fields ↔ UI data model — are exactly the seams a markdown spec with a sequence diagram can pin down. A capability spec that enumerates "mission steps, their states, their failure modes" is *readable by the behaviour engineer and the UX designer and the frontend engineer at once*. That is a real, narrow, achievable convergence — and it happens to be the studio's home territory.

**3. The PM's artefact, upgraded from prose to structure.** §5.2 found the PM already owns the only cross-role artefact and it is non-binding prose. The single highest-leverage move is to give the PM a *structured* spec — proposal/design/tasks with capability specs — that the other roles are *obliged* to read because it is where the agreed behaviour and the agreed failure modes are defined. This makes the PM the natural sponsor of the methodology, which is the most plausible adoption path because the PM is the one role with both the cross-role mandate and the pain.

### 6.3 The honest limits

A lightweight markdown methodology will **not**:

- **Replace the formal artefacts where formality is load-bearing.** The safety engineer's hazard log feeds a certification-body review and a legal PL/SIL sign-off. A markdown spec can *reference* and *summarise* the safety case; it cannot *be* it. Same for the controls engineer's stability proof. The methodology is a connective tissue, not a replacement organ.
- **Capture the controls engineer's or perception engineer's deep work.** State-space models and trained weights are not expressible in markdown and should not be. The methodology touches these roles only at their *interfaces* — "the controller meets this settling-time requirement," "the detector meets this confidence threshold" — and leaves their interiors alone. If the studio tries to pull controls or perception *into* the spec, it repeats the MBSE mistake.
- **Survive if it is heavyweight or mandatory-on-day-one.** The MBSE post-mortem is unambiguous: front-loaded, per-role adoption cost kills unification artefacts. The methodology must be *cheaper to adopt than to ignore* for each role individually. If reading `design.md` takes a UX designer longer than the Slack thread it replaces, it loses.
- **Eliminate the twelve toolchains.** It should not try. The convergence is at the *review surface and the handoff contract*, not the workshop. A methodology that respects the twelve tools and unifies only the seams is doing something genuinely new — because §6.1 shows nobody has — and is modest enough to actually be adopted.

### 6.4 The defensible synthesis

Put plainly: the robotics ecosystem has twelve roles, twelve vocabularies, twelve file formats and twelve definitions of done (§5). Every attempt to unify them has either converged one seam and stopped, or tried for total convergence with a heavyweight model and been rejected (§6.1). The realistic, defensible role for a lightweight spec-driven methodology is **the shared review surface and the handoff contract** — the one artefact all roles read, that pins down intent and failure modes at the seams, sponsored by the PM because the PM already owns the cross-role artefact and already feels the pain (§6.2). The studio should claim exactly that and no more. "We give your twelve specialists one document they all read" is true, achievable and unowned. "We give you one model of the whole robot" is the MBSE claim, and the graveyard is full of it.

---

## Sources & primary references

### Roles, salaries, job definitions
- [Robotics Software Engineer salary — ZipRecruiter](https://www.ziprecruiter.com/Salaries/Robotics-Software-Engineer-Salary)
- [Robotics Software Engineer salary trends — Glassdoor](https://www.glassdoor.com/Salaries/robotics-software-engineer-salary-SRCH_KO0,26.htm)
- [ROS Developer jobs — ZipRecruiter](https://www.ziprecruiter.com/Jobs/Ros-Developer)
- [Robotics Integrator jobs / salary — ZipRecruiter](https://www.ziprecruiter.com/Jobs/Robotics-Integrator)
- [Integration engineer (robotics) jobs — Glassdoor](https://www.glassdoor.com/Job/integration-engineer-robotics-jobs-SRCH_KO0,29.htm)
- [Software Engineer, Robotics Integrations — Tesla careers](https://www.tesla.com/careers/search/job/software-engineer-robotics-integrations-250588)
- [Robotics Control Engineer jobs / salary — ZipRecruiter](https://www.ziprecruiter.com/Jobs/Full-Time-Robotics-Control-Engineer)
- [Introduction to State-Space Control — WPILib docs](https://docs.wpilib.org/en/stable/docs/software/advanced-controls/state-space/state-space-intro.html)
- [Autonomy Engineer job description template — Vintti](https://www.vintti.com/job-description/autonomy-engineer)
- [Introduction to behavior trees — Robohub](https://robohub.org/introduction-to-behavior-trees/)
- [Behavior tree (AI, robotics, control) — Wikipedia](https://en.wikipedia.org/wiki/Behavior_tree_(artificial_intelligence,_robotics_and_control))
- [Robotics Perception Engineer jobs / salary — ZipRecruiter](https://www.ziprecruiter.com/Jobs/Robotics-Perception-Engineer)
- [RobOps — InOrbit](https://www.inorbit.ai/robops)
- [Robot Operations / RobOps key challenges — MOV.AI](https://www.mov.ai/blog/4-keys-to-robot-operations-robops/)
- [Robotic Crew nearshore robotics dev + ops talent — The Robot Report](https://www.therobotreport.com/robotic-crew-offers-nearshore-robotics-development-operations-talent/)
- [NVIDIA Isaac Sim — robotics simulation](https://developer.nvidia.com/isaac/sim)
- [NVIDIA robotics jobs / salary range — ZipRecruiter](https://www.ziprecruiter.com/Jobs/Nvidia-Robotics)
- [ISO 10218-1:2025 Robotics safety requirements — ISO](https://www.iso.org/standard/73933.html)
- [ISO 10218-1:2025 explainer — The ANSI Blog](https://blog.ansi.org/ansi/iso-10218-1-2025-robots-and-robotic-devices-safety/)
- [Robotic Safety Engineer jobs / salary — ZipRecruiter](https://www.ziprecruiter.com/Jobs/Robotic-Safety-Engineer)
- [Robotics and Functional Safety — UL Solutions](https://www.ul.com/resources/robotics-and-functional-safety)
- [Robotics Product Manager jobs / salary — Built In](https://builtin.com/jobs/product/robotics)

### Systems integrators
- [19 Robotic Systems Integrator Manufacturers 2026 — Metoree](https://us.metoree.com/categories/102019/)
- [Robotics Integration & Automation — JR Automation](https://www.jrautomation.com/solutions/technology-integrations/robotics-integration)
- [Integrators — The Robot Report supplier directory](https://www.therobotreport.com/supplier-category/integrators/)

### Robotics UX / HRI
- [What is Human-Robot Interaction (HRI) — Interaction Design Foundation](https://ixdf.org/literature/topics/human-robot-interaction)
- [Human-Robot Interaction Design — Boston Engineering](https://www.boston-engineering.com/solutions/technical-innovation/robotics/robotics-design-and-application-expertise/human-robot-interaction-design/)
- [Career as Human-Robot Interaction Designer — Gladeo](https://losangeles.gladeo.org/career/human-robot-interaction-designer)
- [IIIMPACT — UX product company](https://www.iiimpact.io/)
- [Key Principles of UX Design for Robotic Applications — IIIMPACT](https://www.iiimpact.io/news/key-principles-of-ux-design-for-robotic-applications)
- [Effective UX Testing Methods for Robotic Interfaces — IIIMPACT](https://www.iiimpact.io/news/effective-ux-testing-methods-for-robotic-interfaces)
- [argodesign — product design & innovation firm](https://argodesign.com/)
- [argodesign sells to a $22B IT company — Fast Company](https://www.fastcompany.com/90239194/argodesign-sells-to-a-22-billion-it-company)
- [From Frog to argodesign: Mark Rolston (Apptronik Apollo) — SiliconHills](https://www.siliconhillsnews.com/2024/01/13/from-frog-design-to-argodesign-mark-rolstons-journey-of-innovation-creativity-and-ai-insights/)
- [Punchcut — design & innovation consultancy](https://punchcut.com/)
- [Senior UX Designer, Amazon Industrial Robotics — Amazon Jobs](https://www.amazon.jobs/en/jobs/3172158/senior-ux-designer-amazon-industrial-robotics)
- [Principal UX Design Researcher, Amazon Industrial Robotics — Amazon Jobs](https://www.amazon.jobs/en/jobs/3147766/principal-ux-design-researcher-amazon-industrial-robotics)

### Operator / observability vendors
- [Foxglove Raises $40M Series B — BusinessWire](https://www.businesswire.com/news/home/20251112126106/en/Foxglove-Raises-$40-Million-Series-B-to-Power-the-Future-of-Physical-AI)
- [Foxglove raises $40M to scale data platform — The Robot Report](https://www.therobotreport.com/foxglove-raises-40m-scale-data-platform-roboticists/)
- [Leading the future of robotics infrastructure with Foxglove — Bessemer](https://www.bvp.com/news/leading-the-future-of-robotics-infrastructure-with-foxglove)
- [Formant company profile & funding — Tracxn](https://tracxn.com/d/companies/formant/__57eZfBvFCaCQUMRBcO3pdriFRxYi-3vmPke--QFN7h0)
- [Robot fleet management startup Formant nabs $18M — SiliconANGLE](https://siliconangle.com/2022/01/05/robot-fleet-management-startup-formant-nabs-18m-investment/)
- [InOrbit.AI Secures Series A Funding — BusinessWire](https://www.businesswire.com/news/home/20250930673347/en/InOrbit.AI-Secures-Series-A-Funding-to-Scale-Robot-Orchestration-Platform-Unlocking-a-New-Era-of-Software-Driven-Physical-Workflows)
- [InOrbit Robot Orchestration](https://www.inorbit.ai/multi-vehicle-orchestration)
- [Boston Dynamics Orbit — Robot Fleet Management Software](https://bostondynamics.com/products/orbit/)
- [Boston Dynamics Spot review — price & specs — easy-robots](https://www.easy-robots.com/en/articles/boston-dynamics-spot-review)
- [Transitive Robotics — remote teleop](https://transitiverobotics.com/caps/transitive-robotics/remote-teleop/)
- [Transitive Robotics opens beta of remote operations — The Robot Report](https://www.therobotreport.com/transitive-robotics-opens-beta-of-remote-operations-solution/)
- [Cogniteam — AIoT and Robotics Cloud Platform](https://www.cogniteam.com/)
- [Cogniteam raises $5.6M Series A for Nimbus — Financial IT](https://financialit.net/news/artificial-intelligence/cogniteam-raises-56-million-series-enhance-nimbus-low-code-ai-robotics)
- [Freedom Robotics company profile — PitchBook](https://pitchbook.com/profiles/company/277361-11)
- [Freedom Robotics Fleet Management Platform](https://www.freedomrobotics.com/robotics-fleet-management-platform)
- [Cyberwave — AI-defined robotics platform](https://cyberwave.com/)
- [Milan-based Cyberwave secures €7M — EU-Startups](https://www.eu-startups.com/2025/10/milan-based-cyberwave-secures-e7-million-to-simplify-ai-driven-automation/)
- [DroneDeploy acquires Rocos — Robotics 24/7](https://www.robotics247.com/article/dronedeploy_acquires_rocos_automate_ground_air_jobsite_data_collection)

### Convergence — MBSE, standards, digital twins
- [MBSE adoption experiences: lessons learned — INCOSE / Wiley](https://incose.onlinelibrary.wiley.com/doi/full/10.1002/sys.21717)
- [Challenges of adopting MBSE tools — SpecInnovations](https://specinnovations.com/blog/what-are-the-challenges-of-adopting-mbse-tools)
- [From model to mindset: the real challenge of MBSE — one-sys.eu](https://www.one-sys.eu/en/post/from-model-to-mindset-the-real-challenge-of-model-based-systems-engineering)
- [MBSE for design & integration of complex robotics systems — ResearchGate](https://www.researchgate.net/publication/387663587_Model-Based_Systems_Engineering_MBSE_for_the_Design_and_Integration_of_Complex_Robotics_Systems)
- [ROS-related robotic systems development with MeROS metamodel — arXiv](https://arxiv.org/html/2506.08706v1)
- [What is the MassRobotics AMR Interoperability Standard — MassRobotics](https://www.massrobotics.org/what-is-the-massrobotics-amr-interoperability-standard/)
- [ros_amr_interop bridge (ROS 2 ↔ MassRobotics) — InOrbit / GitHub](https://github.com/inorbit-ai/ros_amr_interop)

### Internal cross-references
- `openspec/_methodology/research/robot-operator-interfaces.md` — companion: platforms, transports, operator-UI vendors (overlaps §4)
- `openspec/_methodology/research/robot-developer-pain-points-and-openspec.md` — companion: where spec-driven dev fits the pain points
- `openspec/current/platform.md` — studio substrate and stack
