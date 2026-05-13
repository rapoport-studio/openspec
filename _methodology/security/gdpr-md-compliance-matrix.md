---
title: GDPR + Moldova Law 133/2011 compliance matrix
status: draft (v1 — batch-1 pass; gap tickets use placeholder RAP-XXX pending Linear epic open)
owner: pavel
audience: [legal, pavel, ai]
tldr: Article-by-article mapping of GDPR and Moldova Law 133/2011 to the studio platform.
      Each gap becomes a Linear sub-issue. Produced as Phase 2 / W2 deliverable of the
      validate-personal-data-security change.
---

# GDPR + Moldova Law 133/2011 compliance matrix

> **Companion to:** [`openspec/archive/2026-05-13-validate-personal-data-security/`](../../archive/2026-05-13-validate-personal-data-security/)
> **Compliance scope elected (P0.2):** Option A — GDPR + Moldova Law 133/2011 only. UK GDPR / AI Act / ISO 27001 unlocked on demand.
> **Status note on gap tickets:** Linear epic not yet open (P0.4 is operator-blocked). Placeholders `RAP-XXX` will be replaced when founder opens the epic and child issues.
> **LINDDUN cross-reference:** Each gap maps to one or more LINDDUN NC-category threats in [`canvas-permission-linddun.md`](canvas-permission-linddun.md).

---

## § 1 GDPR article-by-article matrix

### Principles (Art. 5)

| Article | Requirement | Mechanism in studio | Status | LINDDUN ref | Gap-ticket |
|---|---|---|---|---|---|
| Art. 5(1)(a) lawfulness, fairness, transparency | Process only on a documented lawful basis; tell subjects what is collected and why | No privacy notice exists; no lawful-basis documentation | **Gap** | LP-CF-NC-1, LP-CF-U-1 | RAP-XXX: Author privacy notice + document lawful basis per Art. 6 |
| Art. 5(1)(b) purpose limitation | Data collected for specified purposes; not further processed incompatibly | Canvas messages used for project work + AI coaching — purpose clear but undocumented | **Gap (minor)** | — | RAP-XXX: Document purposes in privacy notice |
| Art. 5(1)(c) data minimisation | Adequate, relevant, limited to what is necessary | `inbox.ip_address` retained indefinitely (90-day anon TODO not implemented); specialist applications retained without expiry | **Gap** | LP-IN-NC-1, LP-SA-NC-1 | RAP-XXX: `add-inbox-ip-anonymization-job`; define retention for `specialist_applications` |
| Art. 5(1)(d) accuracy | Reasonable steps to keep data accurate and up to date | No automated accuracy check; users can edit profile but not DSAR-extracted data | **Gap (minor)** | — | RAP-XXX: Add correction workflow to DSAR runbook |
| Art. 5(1)(e) storage limitation | Kept no longer than necessary; retention schedule documented | No formal retention schedule for any table | **Gap** | LP-OA-NC-1, LP-SA-NC-1 | RAP-XXX: Define per-table retention schedule |
| Art. 5(1)(f) integrity + confidentiality | Appropriate technical + organisational measures | RLS, TLS 1.3, Supabase TDE, magic-link auth, no plaintext secrets in repo | **Satisfied** | — | — |
| Art. 5(2) accountability | Controller able to demonstrate compliance | This matrix + DSAR runbook + breach runbook are the primary evidence artefacts | **Partial (in progress)** | — | Closes when all gap tickets addressed |

### Lawful basis (Art. 6)

| Article | Requirement | Mechanism in studio | Status | Gap-ticket |
|---|---|---|---|---|
| Art. 6(1)(a) consent | Freely given, specific, informed, unambiguous consent | Not used as lawful basis by design (consent is fragile for B2B) | **n/a** | — |
| Art. 6(1)(b) contract | Processing necessary for performance of a contract | Canvas members (owner, co_owner, collaborator) — processing necessary for service delivery | **Satisfied** | — |
| Art. 6(1)(b) contract | External collaborators | No signed DPA / contract with external_collaborator before canvas access | **Gap** | RAP-XXX: Add DPA/service terms for external collaborators before `add-specialist-onboarding` cohort 1 |
| Art. 6(1)(f) legitimate interest | Legitimate interest of controller, not overridden by subject | Inbox leads, specialist applications — legitimate interest basis defensible but must be documented and balanced | **Gap** | RAP-XXX: Document LIA (Legitimate Interest Assessment) for inbox + specialist applications |

### Transparency (Art. 13 + 14)

| Article | Requirement | Mechanism in studio | Status | LINDDUN ref | Gap-ticket |
|---|---|---|---|---|---|
| Art. 13 information at collection | When data collected directly from subject: identity of controller, purposes, legal basis, retention, rights | No privacy notice at any collection point (magic-link, inbox form, specialist form, Telegram onboarding) | **Gap** | LP-CF-U-1, LP-IN-U-1, LP-SA-U-1, LP-TG-U-1 | RAP-XXX: Author and deploy privacy notices for all collection points |
| Art. 14 information where not obtained from subject | When data inferred or collected from third parties (e.g., client mental model built from conversations) | No disclosure to clients that Mirror perception.md is being constructed about their mental model | **Gap** | LP-MR-U-1 | RAP-XXX: Add Mirror disclosure step to client onboarding flow |

### Data subject rights

| Article | Requirement | Mechanism in studio | Status | Gap-ticket |
|---|---|---|---|---|
| Art. 15 access (DSAR) | Subject can request all data held about them; response within 30 days | DSAR runbook authored (this batch); SQL templates and process defined | **Satisfied (after runbook merge)** | — |
| Art. 16 rectification | Subject can request correction of inaccurate data | Studio profile pages allow email correction; Mirror fields correctable by platform_owner on request | **Partial** | RAP-XXX: Define correction workflow for data held outside profile (canvas messages, inbox) |
| Art. 17 erasure ("right to be forgotten") | Subject can request deletion; must cascade to all data | `auth.users CASCADE` deletes linked rows; Mirror files on filesystem require manual deletion; specialist application data requires manual deletion; n8n Telegram data not addressed | **Partial** | RAP-XXX: Complete erasure workflow in DSAR runbook; test CASCADE on synthetic subject |
| Art. 18 restriction of processing | Subject can restrict processing pending accuracy dispute | Not implemented | **Gap (deferred)** | RAP-XXX: Track as future DSAR runbook addition (low priority for current scale) |
| Art. 19 notification obligation | When rectification/erasure done, notify any recipients of the data | Not implemented (no automated notification) | **Gap (minor)** | RAP-XXX: Add to DSAR runbook process |
| Art. 20 data portability | Export data in structured, machine-readable format | `forge mirror-export <slug>` exports Mirror files; no export for canvas messages or other tables | **Partial** | RAP-XXX: Extend DSAR runbook export to include canvas_messages and other tables as JSON |
| Art. 21 right to object | Object to processing based on legitimate interest | Not implemented (no objection mechanism) | **Gap (deferred)** | RAP-XXX: Low priority for current B2B model; add when first consumer-facing product launches |
| Art. 22 automated decision-making | Right not to be subject to solely automated decisions | Muse provides AI recommendations but human (platform_owner) makes all decisions | **Satisfied** | — |

### Privacy by design and security

| Article | Requirement | Mechanism in studio | Status | LINDDUN ref | Gap-ticket |
|---|---|---|---|---|---|
| Art. 25 privacy by design | Default settings must be privacy-friendly; minimum necessary data by default | RLS default-deny; magic-link (no password at rest); no public profiles; BUT Mirror is whole-document-grained — section-level minimisation not implemented | **Partial** | LP-MR-NC-1 | `add-document-zone-permissions` implements section-level minimisation (Art. 25 compliance for Mirror) |
| Art. 32 security measures | Appropriate technical + organisational measures (encryption, pseudonymisation, availability, resilience) | TLS 1.3, Supabase TDE, RLS, Infisical secrets management, Supabase automated backups, Cloudflare DDoS protection | **Satisfied** | — | — |

### Breach notification

| Article | Requirement | Mechanism in studio | Status | Gap-ticket |
|---|---|---|---|---|
| Art. 33 notify regulator within 72h | Notify ANCD (Moldova DPA) within 72 hours of becoming aware of breach | Breach notification runbook authored (this batch) | **Satisfied (after runbook merge)** | — |
| Art. 34 notify subjects | Notify affected data subjects when breach likely to result in high risk | Breach notification runbook § 6 covers subject notification template | **Satisfied (after runbook merge)** | — |

### Data Protection Impact Assessment

| Article | Requirement | Mechanism in studio | Status | Gap-ticket |
|---|---|---|---|---|
| Art. 35 DPIA | Required for high-risk processing (large-scale, sensitive data, systematic monitoring) | Not performed; current scale (< 10 clients) may be below threshold for mandatory DPIA; but Mirror captures mental-model data which is arguably sensitive | **Gap (deferred)** | RAP-XXX: Perform DPIA when first EU-resident enterprise client lands or when active client count exceeds 10 |

### International transfers

| Article | Requirement | Mechanism in studio | Status | Gap-ticket |
|---|---|---|---|---|
| Art. 44–49 international transfer | Adequate protection for transfers outside EEA | Supabase: EU region (eu-central-2); Cloudflare: EU data localisation for Workers; Anthropic: US-based (SCC required); Linear: US-based (SCC required); GitHub: US-based (SCC required); n8n Cloud: EU-hosted | **Partial** | RAP-XXX: Verify and document SCCs with Anthropic, Linear, GitHub as sub-processors; confirm n8n EU hosting |

### Sub-processors (Art. 28)

| Sub-processor | Service | DPA status | Gap-ticket |
|---|---|---|---|
| Supabase | Database, Auth, Storage | Supabase DPA available at supabase.com/privacy — review and accept | **Gap** | RAP-XXX: Accept Supabase DPA formally |
| Cloudflare | Workers, CDN, R2 | Cloudflare DPA available — review and accept | **Gap** | RAP-XXX: Accept Cloudflare DPA formally |
| Anthropic | AI model API | Anthropic DPA available for API customers — review and accept | **Gap** | RAP-XXX: Accept Anthropic DPA; confirm SCC for EU → US transfer |
| Linear | Project management | Linear DPA available — review and accept | **Gap** | RAP-XXX: Accept Linear DPA; confirm SCC |
| GitHub | Source hosting, CI | GitHub DPA (GitHub Customer Agreement) — review and accept | **Gap** | RAP-XXX: Accept GitHub DPA; confirm SCC |
| n8n Cloud | Telegram bridge | n8n DPA available — review and accept; confirm EU hosting | **Gap** | RAP-XXX: Accept n8n DPA; confirm EU-only processing of Telegram messages (LP-TG-NC-1) |
| Infisical | Secrets management | Infisical DPA — review and accept | **Gap** | RAP-XXX: Accept Infisical DPA |

---

## § 2 Moldova Law No. 133/2011 article highlights

> Moldova's data protection law substantially mirrors the GDPR structure (enacted to align with EU acquis). Where Articles align to GDPR counterparts, the GDPR analysis above applies. This section covers Moldova-specific provisions.

| Article | Requirement | GDPR equivalent | Status | Gap-ticket |
|---|---|---|---|---|
| Art. 4–6 definitions + scope | Applies to personal data processed in Moldova or affecting Moldovan residents | GDPR Art. 2–4 | **Satisfied** (studio is Moldovan SRL; GDPR analysis applies) | — |
| Art. 7 lawful basis | Legitimate purpose, adequate means | GDPR Art. 6 | See Art. 6 row above | — |
| Art. 8 sensitive data | Special categories require explicit basis | GDPR Art. 9 | No special-category data processed currently | — |
| Art. 14 subject rights | Access, rectification, erasure, restriction | GDPR Art. 15–21 | See GDPR rows above | — |
| Art. 24 security obligations | Technical + organisational measures | GDPR Art. 32 | **Satisfied** | — |
| Art. 28 notification of ANCD | Notify ANCD of new processing operations / new processors | No GDPR direct equivalent (GDPR replaced mandatory registration with accountability) | **Gap** | RAP-XXX: Notify ANCD of studio processing operations and sub-processors as required under MD law; include Anthropic, n8n as new processors |
| Art. 30 cross-border transfer | Transfer only to adequate countries or under approved mechanism | GDPR Art. 44–49 | See international transfer row above | — |

### Art. 28 ANCD notification — action required

Moldova Law 133/2011 Art. 28 requires the controller (Rapoport Studio SRL) to notify ANCD of the personal data processing operations it conducts, including the categories of data, purposes, and processors. This notification has not been filed.

**Contact:** ANCD (Agenția Națională pentru Protecția Datelor cu Caracter Personal)  
**Website:** datepersonale.md  
**Email for notifications:** `notificare@datepersonale.md` *(verify currency — see breach-notification-runbook.md § 5)*  
**Action:** File initial notification; update when new processor (n8n, Anthropic) added.

---

## § 3 Gap summary by priority

### Critical gaps (must close before next paying client)

1. **No privacy notice at any collection point** (Art. 5(1)(a), Art. 13, Art. 14) — affects all data collection flows.
2. **No lawful basis documented** for external_collaborator processing, inbox leads, specialist applications.
3. **Mirror disclosure** — clients unaware of what data is being processed about them (Art. 14).
4. **n8n DPA + EU hosting confirmation** — Telegram messages transit an undocumented sub-processor.
5. **`inbox.ip_address` indefinite retention** — Art. 5(1)(c) data minimisation violation.

### Major gaps (close within 60 days)

6. All sub-processor DPAs (Supabase, Cloudflare, Anthropic, Linear, GitHub, Infisical).
7. **ANCD notification** under MD Law 133/2011 Art. 28.
8. **Specialist application retention policy** — no defined expiry for rejected applicants.
9. **OAuth token retention policy** for Linear OAuth connections.
10. **Erasure workflow completion** — CASCADE tested; filesystem Mirror files; n8n Telegram data.

### Deferred gaps (trigger-based)

- Art. 35 DPIA — trigger: first EU enterprise client or 10+ active clients.
- Art. 18 restriction of processing — trigger: first formal subject dispute.
- Art. 21 objection — trigger: first consumer-facing product launch.

---

## § 4 Methodology notes

**What constitutes "Satisfied":** The mechanism exists in the platform (RLS, TLS, CASCADE) and has been technically verified (via W5 adversarial audit or equivalent). It does not require documentation alone.

**What constitutes "Partial":** The mechanism exists for some cases but not others, or exists but is untested.

**What constitutes "Gap":** Either the mechanism does not exist, or it exists but is explicitly undocumented and therefore cannot be demonstrated to a regulator.

**Revisit cadence:** This matrix should be updated whenever: (a) a new data collection surface is added, (b) a new sub-processor is engaged, (c) a gap ticket closes, or (d) annually as a compliance check.

---

*Authored: 2026-05-13. Batch-1 pass. Gap ticket IDs are placeholders (RAP-XXX) — to be updated when founder opens the Linear epic (P0.4).*
