---
title: Studio Risk Taxonomy
status: stable
owner: pavel
audience: [pavel, ai, founder, future-operator]
tldr: Five-level risk model (Regulatory → Security → Product → SDD-specific → Implementation) for Rapoport Studio. Each level has a definition, canonical home, owner, and adoption recommendation sourced from ai-risk-frameworks-and-indexing.md §1.4.
---

# Studio Risk Taxonomy

> Source of truth for how risks are categorised and where they live across change proposals, capability specs, and the studio charter.
>
> Living instance of this taxonomy: [`_root/risk-register.md`](../_root/risk-register.md)
> Primary research backing: [`_methodology/research/ai-risk-frameworks-and-indexing.md`](research/ai-risk-frameworks-and-indexing.md)

---

## Overview

Five levels, ordered external → internal. Each level operates at a different cadence and is owned by a different person.

| Level | Name | Frameworks | Adoption |
|---|---|---|---|
| L1 | Regulatory | EU AI Act, GDPR | Partial |
| L2 | Security | OWASP LLM Top 10 2025 | Full |
| L3 | Product | Cagan Four Risks | Full |
| L4 | SDD-specific | NIST AI RMF, MITRE ATLAS (partial), FMEA (epics) | Full + Partial |
| L5 | Implementation | Per-change technical risks | Full |

---

## L1 — Regulatory

**Definition.** External legal and regulatory obligations that constrain how the studio builds and operates AI surfaces. For Rapoport Studio these are primarily the EU AI Act (limited-tier transparency labeling) and GDPR (data minimisation, right to erasure, lawful basis for processing).

**Where it lives.** [`_methodology/studio-charter.md`](studio-charter.md) § Legal + compliance. Regulatory posture is set at the studio level, not per-change.

**Owner.** Pavel Rapoport + legal counsel. AI agents do not decide regulatory classification.

**Adoption recommendation (from research §1.4).** _Partial._ Adopt EU AI Act limited-tier requirements (transparency labeling on AI-generated content: "Drafted by Maestro") and GDPR data-minimisation obligations. Skip high-risk-tier requirements (not applicable to current surfaces) and ISO/IEC 42001 certification (bureaucratic overhead for a 1–3 person studio).

**Review cadence.** Annually or when a new AI surface is introduced (Maestro email, Telegram canvas, in-app AI features).

**Canonical instance — grant compliance (applied 2026-05-13 from change `add-funding-strategy`).** Reporting, audit, and clawback obligations from received grants are a Regulatory risk. Mitigation: per-grant compliance tracking in `funding-pack/applications/<id>/`; accountant review when retained. Owner: Pavel + counsel.

---

## L2 — Security

**Definition.** Application-level security risks specific to LLM-powered systems. The primary checklist is OWASP LLM Top 10 2025: prompt injection (LLM01), sensitive info disclosure (LLM02), supply chain (LLM03), data poisoning (LLM04), improper output handling (LLM05), excessive agency (LLM06), system prompt leakage (LLM07), vector weaknesses (LLM08), misinformation (LLM09), unbounded consumption (LLM10).

**Where it lives.** `_methodology/security/security-review-checklist.md` (Phase 3 — forward reference; file created in RAP-461 Phase 3). Until that file exists, enforce via PR description checklist.

**Owner.** AppSec / Pavel. Every PR touching Forge CLI, Supabase RLS, CF Workers secrets, or auth flows is required to have a manual security review (R7 in [`_root/risk-register.md`](../_root/risk-register.md)).

**Adoption recommendation (from research §1.4).** _Full._ OWASP LLM Top 10 is the direct PR checklist for CLI/Worker code. Adopt MITRE ATLAS Initial Access + Execution tactics partially for Forge threat-model (prompt-injection via repo files — see CVE-2025-53773 note in research file). Skip MITRE ATLAS in full (too heavyweight for one-person security review).

**Review cadence.** Per PR for Forge/auth/RLS changes; quarterly for the checklist itself.

---

## L3 — Product

**Definition.** Classical product risks that apply to every change proposal regardless of technology. Cagan SVPG Four Risks: **Value** (will users pay / use it?), **Usability** (can they figure it out?), **Feasibility** (can we build it in the time-box?), **Business Viability** (does it work for legal / finance / brand?).

**Where it lives.** `proposal.md § Risks` — mandatory section in every change proposal. Each of the four risks must be explicitly addressed (one line minimum; "not applicable" with justification is acceptable).

**Owner.** PM / Pavel for Value + Viability; change author for Feasibility + Usability.

**Adoption recommendation (from research §1.4).** _Full._ Cagan Four Risks is the proposal template standard. No change proposal ships without addressing all four. RAID log (Asana model) is handled at the epic level by Linear — not required in individual proposals.

**Review cadence.** Per change proposal at draft time.

**Canonical instance — positioning trajectory.** Stage-transition failure (a drift trigger from `_methodology/positioning-decision-framework.md § 4` firing without corrective action) is a Business Viability risk: the studio builds the right product but fails to reach the market stage where it becomes viable. Registered at archive of `lock-positioning-trajectory` (2026-05-13).

**Canonical instance — funding viability (applied 2026-05-13 from change `add-funding-strategy`).** Runway exhaustion before validated wedge is a Business Viability risk. Mitigation: `_methodology/funding-strategy.md` five-channel diversification; quarterly review (D-FUND-7). Owner: Pavel.

---

## L4 — SDD-specific

**Definition.** Risks that emerge specifically from Specification-Driven Development with AI agents. Ten categories (from research Top-10 SDD table):

1. **Spec-implementation drift** — spec update must land in same PR as code (R1 in risk-register).
2. **Hallucination in spec-generation** — ground all AI-authored specs with `<read_first>` files; ban new identifiers without a source reference.
3. **Stale documentation cascade** — refs guard in CI; status lifecycle enforced.
4. **Cascading errors** — pre-merge verifier gate; fail-fast in epic chains. (_CodeRabbit 2025: AI PRs carry 1.7× more issues than human PRs._)
5. **Ambiguity in spec language** — Ubiquitous Language; banned subjective qualifiers (fast / robust / user-friendly).
6. **Lost context / context window overflow** — phase split mandatory if >4 files touched (R4 in risk-register); bundle pattern for Forge.
7. **Prompt injection via repo files** — Forge never auto-executes instructions from non-trusted markdown; sandbox file reading.
8. **Bias amplification through templates** — periodic template audit; diverse few-shot exemplars.
9. **Tooling single-point-of-failure (SPOF)** — multi-provider failover deferred to $10K+/mo Anthropic spend threshold; tracked as assumption A1 in risk-register.
10. **SpecFall antipattern** — `design.md` hard cap 2000 words; longer = split change proposal (R5 in risk-register).

**Where it lives.** `proposal.md § Risks` (high-level, change-specific SDD risks) + Forge verifier (automated gates: refs-guard, type-check, lint, test). The cross-epic SDD risk log lives in [`_root/risk-register.md`](../_root/risk-register.md).

**Owner.** AI agent (automated gates) + Pavel (design-time identification and verifier configuration).

**Adoption recommendation (from research §1.4).** _Full_ for drift / hallucination / ambiguity / cascading errors / SpecFall (these are structural, always-on). _Partial_ for FMEA: apply RPN scoring only for epics > 5 PRs (RPN > 200 = blocking; must have automated mitigation). _Partial_ for tooling SPOF: lightweight (note the risk in risk-register); full mitigation deferred.

**Review cadence.** Drift + SpecFall — per PR. SDD risk-register entries — quarterly grep-audit (see R1 in risk-register).

**Research cross-reference.** [`research/spec-to-code-validity-2026.md`](research/spec-to-code-validity-2026.md) — drift measurement methodology. [`research/trust-gradient-evolution-2023-2026.md`](research/trust-gradient-evolution-2023-2026.md) — which task categories are structurally hard for AI agents.

---

## L5 — Implementation

**Definition.** Per-change technical risks: edge cases, concurrency hazards, performance cliffs, backward-compatibility breaks, migration risks, data-loss scenarios. Scope is a single Linear issue and its associated PR.

**Where it lives.** `design.md § Risks` — required section in every design document. At minimum: list the top-3 implementation risks and their mitigations.

**Owner.** Change author (human or AI). Pavel reviews for HIGH-severity items (concurrency, RLS, secrets — see R2, R7 in risk-register).

**Adoption recommendation (from research §1.4).** _Full._ Every design.md must have a Risks section. For Forge/auth/RLS changes, escalate to L2 Security review simultaneously.

**Review cadence.** Per change at design time; re-evaluated at PR review if scope changed.

---

## Privacy-disclosure risk category (applied 2026-05-13 from change `validate-personal-data-security`)

**Definition.** Risks arising from unintended exposure of personally identifiable information across the studio's data surfaces — canvas content, Mirror fields, user metadata, and backup/export paths. Distinct from L2 Security (application-level exploits) and L1 Regulatory (legal obligations): Privacy-disclosure covers architectural privacy threats that may not violate a law but erode user trust and violate the studio's data-minimisation ethos.

**Framework.** LINDDUN GO (7-category privacy threat model): Linkability, Identifiability, Non-repudiation, Detectability, Disclosure of information, Unawareness, Non-compliance. Exemplar artefact: `_methodology/security/canvas-permission-linddun.md`.

**Where it lives.** For capabilities with PII surface: a `## LINDDUN GO summary` section in `design.md` (mandatory when trigger criteria in `threat-model-template.md § LINDDUN GO requirement` are met). Full threat catalogue + matrix artefact for novel surfaces.

**Owner.** Change author + Pavel review.

**Trigger.** See `_methodology/security/threat-model-template.md § LINDDUN GO requirement` for the enumerated trigger conditions.

**Review cadence.** Per change when triggered. Cross-cutting privacy review annually or when new PII processor added.

---

## Compliance non-conformance risk category (applied 2026-05-13 from change `validate-personal-data-security`)

**Definition.** Risks of failing to satisfy specific regulatory article obligations (GDPR, Moldova Law 133/2011) even when the application is technically secure. Examples: DSAR response exceeding 30-day window, breach notification not filed within 72 hours, no lawful basis documented for a new data collection.

**Framework.** GDPR + MD-133 compliance matrix — article-by-article gap tracking. Artefact: `_methodology/security/gdpr-md-compliance-matrix.md` (in `archive/2026-05-13-validate-personal-data-security/`).

**Where it lives.** Per-change: `design.md § Risks` compliance row when the change touches personal data handling. Studio-level: `_methodology/security/gdpr-md-compliance-matrix.md` tracks gap status across articles.

**Owner.** Pavel + legal counsel (counsel required for breach notification template letters and DPIA).

**Review cadence.** Per-change when PII surface touched. Gap status reviewed annually or when new processor engaged.

---

## Privacy-disclosure risk category (applied 2026-05-13 from change `validate-personal-data-security`)

**Definition.** Risks arising from unintended exposure of personally identifiable information across the studio's data surfaces — canvas content, Mirror fields, user metadata, and backup/export paths. Distinct from L2 Security (application-level exploits) and L1 Regulatory (legal obligations): Privacy-disclosure covers architectural privacy threats that may not violate a law but erode user trust and violate the studio's data-minimisation ethos.

**Framework.** LINDDUN GO (7-category privacy threat model): Linkability, Identifiability, Non-repudiation, Detectability, Disclosure of information, Unawareness, Non-compliance. Exemplar artefact: `_methodology/security/canvas-permission-linddun.md`.

**Where it lives.** For capabilities with PII surface: a `## LINDDUN GO summary` section in `design.md` (mandatory when trigger criteria in `threat-model-template.md § LINDDUN GO requirement` are met). Full threat catalogue + matrix artefact for novel surfaces.

**Owner.** Change author + Pavel review.

**Trigger.** See `_methodology/security/threat-model-template.md § LINDDUN GO requirement` for the enumerated trigger conditions.

**Review cadence.** Per change when triggered. Cross-cutting privacy review annually or when new PII processor added.

---

## Compliance non-conformance risk category (applied 2026-05-13 from change `validate-personal-data-security`)

**Definition.** Risks of failing to satisfy specific regulatory article obligations (GDPR, Moldova Law 133/2011) even when the application is technically secure. Examples: DSAR response exceeding 30-day window, breach notification not filed within 72 hours, no lawful basis documented for a new data collection.

**Framework.** GDPR + MD-133 compliance matrix — article-by-article gap tracking. Artefact: `_methodology/security/gdpr-md-compliance-matrix.md` (in `archive/2026-05-13-validate-personal-data-security/`).

**Where it lives.** Per-change: `design.md § Risks` compliance row when the change touches personal data handling. Studio-level: `_methodology/security/gdpr-md-compliance-matrix.md` tracks gap status across articles.

**Owner.** Pavel + legal counsel (counsel required for breach notification template letters and DPIA).

**Review cadence.** Per-change when PII surface touched. Gap status reviewed annually or when new processor engaged.

---

## Cross-references

| Resource | Purpose |
|---|---|
| [`_root/risk-register.md`](../_root/risk-register.md) | Living log of open cross-epic risks; entries cite taxonomy level |
| [`research/ai-risk-frameworks-and-indexing.md`](research/ai-risk-frameworks-and-indexing.md) | Source research: framework comparison, adoption recommendation rationale, SDD top-10 |
| [`research/spec-to-code-validity-2026.md`](research/spec-to-code-validity-2026.md) | Drift measurement methodology (L4 reference) |
| [`research/trust-gradient-evolution-2023-2026.md`](research/trust-gradient-evolution-2023-2026.md) | Which categories remain structurally hard for AI (L4 + L2 reference) |
| `_methodology/security/security-review-checklist.md` | L2 OWASP checklist (Phase 3 forward reference) |
| `_methodology/studio-charter.md` | L1 regulatory posture |
| `_methodology/security/canvas-permission-linddun.md` | Privacy-disclosure LINDDUN GO exemplar artefact |
| `archive/2026-05-13-validate-personal-data-security/` | Source change: W1–W6 security artefacts, LINDDUN GO, GDPR+MD-133 matrix, DSAR runbook, breach notification runbook, adversarial RLS audit |
| `_methodology/security/canvas-permission-linddun.md` | Privacy-disclosure LINDDUN GO exemplar artefact |
| `archive/2026-05-13-validate-personal-data-security/` | Source change: W1–W6 security artefacts, LINDDUN GO, GDPR+MD-133 matrix, DSAR runbook, breach notification runbook, adversarial RLS audit |

---

## What is out of scope

- **ISO/IEC 42001 certification** — bureaucratic overhead for a 1–3 person studio; revisit at 10-person scale.
- **Full STRIDE-per-element threat models** — lightweight Shostack 4Q is sufficient; formal STRIDE deferred.
- **Anthropic RSP / ASL as rulebook** — frontier-lab concept; useful as worldview only, not binding for the studio.
- **EU AI Act high-risk tier requirements** — not applicable (no credit-scoring, biometric, or critical-infrastructure surfaces).
- **Full RAID log per epic** — Linear issues serve this purpose; this taxonomy governs the cross-epic risk-register only.
- **Quarterly red-lines automation** — tracked in risk-register; manual quarterly review until automation is justified (Phase 4).
- **Mutation testing (Stryker)** — tracked as R3 in risk-register; deferred to Epic 7.
