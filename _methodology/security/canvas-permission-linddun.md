---
title: Canvas permission LINDDUN threat model
status: draft (v1 — batch-1 pass; founder review pending)
owner: pavel
audience: [security-specialist, pavel, ai]
tldr: Privacy-focused threat model for the canvas access surface. Applies the seven
      LINDDUN GO categories (Linkability, Identifiability, Non-repudiation,
      Detectability, Disclosure of information, Unawareness, Non-compliance) to
      each of the ten data flows that touch personal data in the studio platform.
      Produced as Phase 1 / W1 deliverable of the validate-personal-data-security change.
---

# Canvas permission LINDDUN threat model

> **Companion to:** [`openspec/changes/validate-personal-data-security/`](../../changes/validate-personal-data-security/)
> **LINDDUN scope elected (P0.1):** Option A — LINDDUN GO (lightweight 7-category × 10-flow matrix).
> **PRO escalation trigger (D-VS-5):** Any single threat classified Critical with no architectural mitigation available. No such threat found in v1; PRO NOT triggered.
> **Status:** Draft. Founder review pending (operator-blocked).

---

## § 0 Scope + assumptions

**In scope:** Every data flow in the studio platform that writes, reads, or transmits *personal data* as defined by GDPR Art. 4(1) — any information relating to an identified or identifiable natural person. This includes direct subjects (platform members) and third parties named in canvas content.

**Out of scope:** Supabase platform internals, Cloudflare platform internals, Anthropic model weights. These are sub-processor responsibilities governed by their respective DPAs.

**Roles in scope (per `redesign-canvas-permission-model § 3`):**
- `platform_owner` — Pavel (sole current instance)
- `owner` — canvas creator; typically a client
- `co_owner` — trusted collaborator with full canvas access
- `collaborator` — internal studio member with project access
- `external_collaborator` — third-party specialist (cohort 1 of `add-specialist-onboarding`)
- `viewer` — read-only canvas member
- `client` — the engagement client; restricted to their own canvas
- `stranger` — authenticated user with no canvas membership; unauthenticated visitor

**Trust boundary model:** Three concentric trust zones:
1. **Platform zone** — `platform_owner` only; sees all data via service-role
2. **Canvas zone** — members of a given canvas; RLS-isolated per `canvas_invitees`
3. **Public zone** — unauthenticated visitors; static renders only

**Assumptions:**
- RLS is the primary isolation mechanism; trust in its correctness is validated by W5 adversarial audit (`canvas-rls-adversarial-audit-2026.md`).
- Magic-link auth prevents credential-stuffing; no password brute-force surface.
- All transport is TLS 1.3; network-layer eavesdropping is out of scope.
- Supabase TDE covers storage-at-rest; physical storage side-channels are out of scope.

---

## § 1 Data flow inventory

Ten flows sourced from existing specs. Each flow is classified, trust-boundaries identified, and spec citation noted.

| ID | Flow name | Source → Destination | Data classification | Trust boundaries crossed | Spec citation |
|---|---|---|---|---|---|
| **CF-CM** | Canvas chat message | User browser → `canvas_messages` (Postgres via Supabase REST) | Confidential — may contain mental models, sales intelligence, personal judgements | User browser → Supabase public API (JWT-authenticated) | `openspec/current/canvas.md` |
| **CF-MR** | Mirror authoring (`write_mirror_field`) | Muse tool call → `canvas_files` path `openspec/mirror/<slug>/perception.md` | Restricted / Confidential — founder's private interpretation; mental model; vocabulary | Muse (server action) → Supabase storage + DB | `openspec/current/mirror.md` |
| **CF-RG** | Render generation | `canvas_files` → static HTML in `openspec/renders/<slug>/` | Internal / Confidential — render includes any Mirror field served to its declared audience | Forge CLI → Cloudflare R2 / Pages | `openspec/current/mirror.md § Render pipeline` |
| **CF-TU** | Tool execution audit | Server actions → `canvas_tool_uses` | Internal — tool name, parameters, user identity | Muse tool handler → Postgres | existing schema |
| **CF-TG** | Telegram surface | Telegram API → `canvas_proposals`, `tg_user_bindings`, `channels` | Confidential — proposal content, Telegram identity binding | Telegram servers → Workers → Supabase | `openspec/changes/add-tg-canvas-surface/` |
| **CF-OA** | OAuth token storage | OAuth callback → `linear_oauth_connections` | Restricted — access token; can impersonate user on Linear | OAuth provider → Studio callback → Postgres | `openspec/changes/add-linear-oauth-connection/` |
| **CF-IN** | Inbox lead capture | Public form → `inbox` | Confidential — name, email, IP address, business description | Public internet → Supabase public insert | `openspec/changes/add-client-onboarding/` |
| **CF-SA** | Specialist application | Application form → `specialist_applications` | Confidential — professional background, contact details, work samples | Public internet → Supabase public insert | `openspec/changes/add-specialist-onboarding/` |
| **CF-ML** | Magic-link sign-in | Email magic link → `auth.users` materialisation | Restricted — email address; auth token in URL | Email provider → Browser → Supabase Auth | Supabase Auth docs |
| **CF-PC** | Cross-canvas project connection | `project_connections.config` (credential fields) → consuming canvas | Restricted — API keys, webhook secrets, integration credentials | Forge service-role read → canvas viewer read path | `openspec/changes/add-project-connections/` |

---

## § 2 LINDDUN GO category × data flow matrix

Each cell is either `n/a` or a list of threat IDs. Empty cells indicate no threat identified in this category for this flow.

| Category | CF-CM | CF-MR | CF-RG | CF-TU | CF-TG | CF-OA | CF-IN | CF-SA | CF-ML | CF-PC |
|---|---|---|---|---|---|---|---|---|---|---|
| **L** Linkability | LP-CM-L-1 | n/a | LP-RG-L-1 | LP-TU-L-1 | LP-TG-L-1 | n/a | LP-IN-L-1 | n/a | n/a | n/a |
| **I** Identifiability | n/a | LP-MR-I-1 | n/a | n/a | LP-TG-I-1 | n/a | LP-IN-I-1 | LP-SA-I-1 | n/a | n/a |
| **N** Non-repudiation | LP-CM-N-1 | n/a | n/a | LP-TU-N-1 | n/a | n/a | n/a | n/a | n/a | n/a |
| **D** Detectability | LP-CM-D-1 | LP-MR-D-1 | n/a | n/a | LP-TG-D-1 | LP-OA-D-1 | n/a | n/a | n/a | n/a |
| **DD** Disclosure | LP-CM-DD-1 | LP-MR-DD-1 | LP-RG-DD-1 | n/a | n/a | LP-OA-DD-1 | n/a | n/a | n/a | LP-PC-DD-1 |
| **U** Unawareness | LP-CF-U-1 | LP-MR-U-1 | n/a | n/a | LP-TG-U-1 | n/a | LP-IN-U-1 | LP-SA-U-1 | n/a | n/a |
| **NC** Non-compliance | LP-CM-NC-1 | LP-MR-NC-1 | n/a | n/a | LP-TG-NC-1 | LP-OA-NC-1 | LP-IN-NC-1 | LP-SA-NC-1 | n/a | LP-PC-NC-1 |

**Category key:**
- **L** Linkability — two data items that should be unlinkable can be joined
- **I** Identifiability — pseudonymous data becomes re-identifiable through combination
- **N** Non-repudiation — subject cannot plausibly deny an attributed action
- **D** Detectability — existence of a record reveals sensitive information
- **DD** Disclosure of information — data exposed to unauthorised actors
- **U** Unawareness — subject unaware of what is collected or how it is used
- **NC** Non-compliance — system does not satisfy stated policies or regulations

---

## § 3 Threat catalogue

27 threats identified across all 7 LINDDUN categories. Row shape per design.md § 3.4.

### L — Linkability threats

| ID | Category | Data flow | Description | Likelihood | Impact | Mitigation (existing) | Mitigation (proposed) | Status |
|---|---|---|---|---|---|---|---|---|
| LP-CM-L-1 | Linkability | Canvas chat | Anonymous inbox lead (email only) can be linked to a `canvas_messages` author if both share an email address, exposing that the lead converted to a client | Low | Medium | `inbox` and `canvas_messages` are separate tables; no automatic join exposed to any role except `platform_owner` | None additional; accept-by-design for `platform_owner` | Closed (accepted) |
| LP-RG-L-1 | Linkability | Render generation | Render slug is derived from project slug; guessing the slug links a public render to a known client | Low | Medium | Render slugs are not publicly indexed; serve under authenticated path or obfuscated R2 URL | Add slug obfuscation or signed URLs for renders | Open → `add-document-zone-permissions` |
| LP-TU-L-1 | Linkability | Tool execution audit | `canvas_tool_uses` rows carry `canvas_id` + `user_id`; a compromised `platform_owner` session links every user's AI interactions | Low | High | `platform_owner` is sole, single-instance role; no delegation | Document in risk-taxonomy as accepted `platform_owner` concentration risk | Closed (accepted) |
| LP-TG-L-1 | Linkability | Telegram surface | Telegram `chat_id` in `tg_user_bindings` can be cross-referenced with public Telegram profile to de-anonymise a `viewer` or `client` | Low | Medium | `tg_user_bindings` is RLS-isolated; only `platform_owner` has select-all | Accept; note in privacy notice that Telegram identity is stored | Closed (accepted) |
| LP-IN-L-1 | Linkability | Inbox lead | `inbox.ip_address` retained indefinitely allows linking multiple leads from same IP (e.g. competitors, aggregators) | Medium | Medium | 90-day anon TODO in `db/entities.md` not yet implemented | Implement `add-inbox-ip-anonymization-job` | **Open → gap** |

### I — Identifiability threats

| ID | Category | Data flow | Description | Likelihood | Impact | Mitigation (existing) | Mitigation (proposed) | Status |
|---|---|---|---|---|---|---|---|---|
| LP-MR-I-1 | Identifiability | Mirror authoring | `perception.md § Mental model` contains founder-private interpretation of client; if client ever reads the Mirror file directly they are re-identifiable as the subject of the model | Low | High | Mirror files are in `canvas_files` behind RLS; `client` role has read access scoped to their own canvas only | Audience-tag `## Mental model` as `[owner, co_owner, collaborator]` per W4 recommendation; blocks client read path | Open → `add-document-zone-permissions` |
| LP-TG-I-1 | Identifiability | Telegram surface | A user who joined Telegram anonymously and also submitted an inbox lead shares an email; combined record is re-identifiable | Low | Low | Separate code paths; no automatic join | None; document in privacy notice | Closed (accepted) |
| LP-IN-I-1 | Identifiability | Inbox lead | Name + email + business description together are directly identifiable; IP address adds location dimension | High | High | No mitigation; by design (needed for DSAR) | Keep but implement retention schedule + Art. 13 notice at form submission | Open → GDPR gap |
| LP-SA-I-1 | Identifiability | Specialist application | Name + LinkedIn + work samples + email form a uniquely identifiable profile; retained without defined expiry | Medium | High | No automated expiry | Define retention schedule for `specialist_applications`; add Art. 13 notice | Open → GDPR gap |

### N — Non-repudiation threats

| ID | Category | Data flow | Description | Likelihood | Impact | Mitigation (existing) | Mitigation (proposed) | Status |
|---|---|---|---|---|---|---|---|---|
| LP-CM-N-1 | Non-repudiation | Canvas chat | `canvas_messages` rows carry `created_by` and are immutable; a user cannot deny having sent a message, even if it was sent in error or if context changed | Low | Low | By design — canvas messages are the accountability record for the engagement | Accept; document in ToS / privacy notice that messages are retained | Closed (accepted) |
| LP-TU-N-1 | Non-repudiation | Tool execution | `canvas_tool_uses` rows bind user identity to every AI tool invocation; user cannot deny a tool was called on their behalf | Low | Low | By design — tool audit trail is a security requirement | Accept | Closed (accepted) |

### D — Detectability threats

| ID | Category | Data flow | Description | Likelihood | Impact | Mitigation (existing) | Mitigation (proposed) | Status |
|---|---|---|---|---|---|---|---|---|
| LP-CM-D-1 | Detectability | Canvas chat | The existence of a canvas with a given client is itself sensitive ("client X has an engagement with the studio"). An `external_collaborator` who can see the canvas can infer the engagement exists. | Medium | Medium | Access restricted to canvas members only; `external_collaborator` must be invited | Accept for invited collaborators; document in onboarding disclosure | Closed (accepted) |
| LP-MR-D-1 | Detectability | Mirror authoring | `canvas_files` path `openspec/mirror/<slug>/` reveals the Mirror slug; slug may match client's company name | Low | Low | R2 paths not publicly listed | None additional | Closed (accepted) |
| LP-TG-D-1 | Detectability | Telegram surface | The existence of a `tg_user_bindings` row for a user reveals they use Telegram; detectability risk for privacy-conscious users | Low | Low | Table RLS-isolated | Accept | Closed (accepted) |
| LP-OA-D-1 | Detectability | OAuth token | Existence of `linear_oauth_connections` row reveals a user has connected Linear; exposes which tools they use | Low | Low | Table RLS-isolated; only canvas members with relevant access can detect | Accept | Closed (accepted) |

### DD — Disclosure of information threats

| ID | Category | Data flow | Description | Likelihood | Impact | Mitigation (existing) | Mitigation (proposed) | Status |
|---|---|---|---|---|---|---|---|---|
| LP-CM-DD-1 | Disclosure | Canvas chat | `external_collaborator` invited to project A could, if RLS has gap, read messages from project B's canvas | Low | Critical | Cross-canvas RLS isolation: `canvas_messages.canvas_id` tied to `canvas_invitees` membership | W5 adversarial audit validates this; no known gap | Closed (pending W5 first run) |
| LP-MR-DD-1 | Disclosure | Mirror authoring | An `external_collaborator` hired into project A reads `§ Mental model` of project B's perception.md by guessing the slug | Low | High | Per-canvas RLS on `canvas_files` + cross-slug isolation per `mirror.md § Visibility model` | None additional; existing mitigation adequate | Closed |
| LP-RG-DD-1 | Disclosure | Render generation | A render at a guessable URL exposes Mirror content to any authenticated user, not just canvas members | Low | Medium | Renders currently served under authenticated path | Verify in W6 render side-channel review | Closed (pending W6) |
| LP-OA-DD-1 | Disclosure | OAuth token | `linear_oauth_connections.access_token` exposed to any `platform_owner` session; token allows impersonation on Linear | Low | Critical | `platform_owner` is sole, single-instance role; token stored encrypted at rest via Supabase TDE | Token should be stored encrypted at application layer; tracked as gap | **Open → gap** |
| LP-PC-DD-1 | Disclosure | Project connections | `project_connections.config` contains API keys / webhook secrets; `viewer` role receives credential-elided projection per D-PM-4 design in `redesign-canvas-permission-model` | Low | Critical | D-PM-4 credential elision not yet designed; W6 side-channel review must complete first | Complete D-PM-4 in `redesign-canvas-permission-model` before viewer role ships | Open → `redesign-canvas-permission-model` D-PM-4 |

### U — Unawareness threats

| ID | Category | Data flow | Description | Likelihood | Impact | Mitigation (existing) | Mitigation (proposed) | Status |
|---|---|---|---|---|---|---|---|---|
| LP-CF-U-1 | Unawareness | All flows | No privacy notice at any collection point; subjects don't know what is collected, for how long, or their rights | High | High | None | Author and deploy privacy notices (Art. 13/14) | **Open → GDPR gap** |
| LP-MR-U-1 | Unawareness | Mirror authoring | Client is not informed that a `perception.md` file containing the studio's interpretation of their mental model is being maintained | High | High | No disclosure mechanism | Add Mirror disclosure step to client onboarding flow | **Open → GDPR gap** |
| LP-TG-U-1 | Unawareness | Telegram surface | Users who interact via Telegram bot don't know their Telegram identity is persisted in `tg_user_bindings` | Medium | Medium | None | Add disclosure to Telegram bot onboarding flow | **Open → GDPR gap** |
| LP-IN-U-1 | Unawareness | Inbox lead | No Art. 13 notice at inbox form submission | High | High | None | Add privacy notice to inbox form | **Open → GDPR gap** |
| LP-SA-U-1 | Unawareness | Specialist application | No Art. 13 notice at specialist application form | High | High | None | Add privacy notice to specialist application form | **Open → GDPR gap** |

### NC — Non-compliance threats

| ID | Category | Data flow | Description | Likelihood | Impact | Mitigation (existing) | Mitigation (proposed) | Status |
|---|---|---|---|---|---|---|---|---|
| LP-CM-NC-1 | Non-compliance | Canvas chat | No documented retention period for `canvas_messages`; Art. 5(1)(e) storage limitation not met | High | High | None | Define per-table retention schedule | **Open → GDPR gap** |
| LP-MR-NC-1 | Non-compliance | Mirror authoring | `perception.md` built without Art. 14 disclosure to the client-as-subject | High | Medium | None | Art. 14 notice in onboarding flow | **Open → GDPR gap** |
| LP-TG-NC-1 | Non-compliance | Telegram surface | `tg_user_bindings` lacks documented lawful basis and retention period | Medium | Medium | None | Document lawful basis; add to privacy notice; define retention | **Open → GDPR gap** |
| LP-OA-NC-1 | Non-compliance | OAuth token | No DPA with Linear Inc. as a sub-processor documented in the repo | Medium | Medium | None | Draft and sign Linear DPA; reference from compliance matrix | **Open → GDPR gap** |
| LP-IN-NC-1 | Non-compliance | Inbox lead | `inbox.ip_address` retained indefinitely; Art. 5(1)(c) data minimisation violated | High | Medium | 90-day anon TODO in schema (not implemented) | Implement `add-inbox-ip-anonymization-job` | **Open → GDPR gap** |
| LP-SA-NC-1 | Non-compliance | Specialist application | `specialist_applications` retained without expiry; Art. 5(1)(e) violated | Medium | Medium | None | Define retention schedule | **Open → GDPR gap** |
| LP-PC-NC-1 | Non-compliance | Project connections | No DPA with connected integration providers (Linear, GitHub etc.) documented for data transmitted through connections | Medium | Low | None | Document sub-processor DPAs | **Open → GDPR gap** |

---

## § 4 Mitigation mapping

Summary of mitigation status by threat. Threats with no mitigation become Phase 7 open questions or feed `add-document-zone-permissions`.

**Existing mitigations (structural):**
- RLS default-deny on all tables containing personal data
- Magic-link authentication (no password brute-force surface)
- TLS 1.3 on all transport paths
- Supabase TDE for storage at rest
- Canvas isolation via `canvas_invitees` membership gate

**Proposed mitigations (not yet implemented):**
1. Audience tagging for Mirror sections → `add-document-zone-permissions` (LP-MR-I-1, LP-RG-L-1)
2. Inbox IP anonymisation job → `add-inbox-ip-anonymization-job` (LP-IN-L-1, LP-IN-NC-1)
3. Privacy notices at all collection points → tracked as GDPR gap-tickets (LP-CF-U-1, LP-MR-U-1, LP-TG-U-1, LP-IN-U-1, LP-SA-U-1)
4. Retention schedules for all tables → tracked as GDPR gap-tickets (LP-CM-NC-1, LP-SA-NC-1)
5. Application-layer OAuth token encryption → tracked as gap-ticket (LP-OA-DD-1)
6. D-PM-4 credential elision for viewer role → `redesign-canvas-permission-model` (LP-PC-DD-1)

**LINDDUN PRO escalation check (D-VS-5):** No threat in this catalogue is classified Critical with *zero* architectural mitigation. LP-CM-DD-1 (cross-canvas disclosure) is Critical-potential but mitigated by RLS isolation; validation is pending W5 first run. LP-PC-DD-1 is Critical-potential but blocked on D-PM-4 design, which is in progress. **PRO NOT triggered for v1.**

---

## § 5 Open questions

Six open questions extracted from § 4. Each becomes a Linear sub-issue or feeds `add-document-zone-permissions`.

| OQ-ID | Question | Source threat(s) | Resolution path |
|---|---|---|---|
| OQ-1 | Does W5 first run confirm zero cross-canvas RLS gaps for all role combinations? | LP-CM-DD-1 | W5 adversarial audit; first run against preview Supabase (operator-blocked) |
| OQ-2 | Does W4 audience-tagging recommendation (`add-document-zone-permissions`) cover all Mirror fields that carry personal data? | LP-MR-I-1, LP-MR-U-1 | W3+W4 content audit + per-section mapping in `document-zone-mechanisms.md` |
| OQ-3 | Should OAuth tokens (Linear, future GitHub) be encrypted at the application layer in addition to Supabase TDE? | LP-OA-DD-1 | Security review by founder; tracked as gap-ticket RAP-XXX |
| OQ-4 | What is the correct retention schedule for each table containing personal data? | LP-CM-NC-1, LP-SA-NC-1, LP-TG-NC-1 | Phase 2 W2 compliance matrix closes this; tracked as GDPR gap-tickets |
| OQ-5 | Does the credential-elision projection for `viewer` role fully prevent LP-PC-DD-1? | LP-PC-DD-1 | `redesign-canvas-permission-model` D-PM-4; W6 side-channel review |
| OQ-6 | Are Telegram-sourced proposals retained longer than necessary under Art. 5(1)(e)? | LP-TG-NC-1 | Phase 2 compliance matrix; Telegram DPA review |

---

## § 6 Cross-references

| Artefact | Relationship |
|---|---|
| `openspec/changes/validate-personal-data-security/design.md` | Owns the framework selection rationale and artefact shape specification |
| `openspec/_methodology/security/gdpr-md-compliance-matrix.md` | NC-category threats map to specific GDPR articles in that matrix |
| `openspec/_methodology/security/canvas-rls-adversarial-audit-2026.md` | W5 adversarial audit validates DD-category closed threats |
| `openspec/_methodology/security/side-channel-audit.md` | W6 side-channel review validates DD and D-category closed threats |
| `openspec/_methodology/research/document-zone-mechanisms.md` | W3+W4 recommendation resolves LP-MR-I-1 and LP-RG-L-1 |
| `openspec/_methodology/research/access-and-context-hierarchy-2026.md` | Prior art on RBAC vs CapBAC; confused-deputy and cache-poisoning incident catalogue |
| `openspec/current/mirror.md` | CF-MR and CF-RG flow definitions |
| `openspec/current/canvas.md` | CF-CM flow definition |
| `openspec/changes/redesign-canvas-permission-model/` | Role hierarchy this threat model validates against |
| `openspec/changes/add-document-zone-permissions/` | Consumes W4 audience-tagging recommendation |
| `openspec/changes/add-specialist-onboarding/` | CF-SA flow; external_collaborator trust boundary introduction |

---

## § 7 Worked example — Mirror authoring flow (CF-MR)

> Full LINDDUN GO analysis for the Mirror authoring flow, per design.md § 3.5. This is the highest-PII-density flow in the platform.

**Flow description:** When Muse invokes `write_mirror_field`, it writes structured data into `canvas_files` at the path `openspec/mirror/<project-slug>/perception.md`. The file contains the studio's accumulated understanding of the client: vocabulary, mental model, comparable products, forbidden patterns, success phrase, aspirational tone. This file is:
- Created by: Muse (server action) on behalf of `platform_owner` or `collaborator`
- Readable by: canvas members with `canvas_files` read access per RLS
- Contains: highly sensitive personal data — founder's private interpretations, inferred psychological/business state

**L — Linkability:** The Mirror slug equals the project slug, which often equals or contains the client's company name. A `collaborator` with access to project A who learns the naming convention could attempt to guess project B's Mirror slug. **Mitigation:** Cross-canvas RLS on `canvas_files` prevents reads outside membership. Slug guessing produces 403, not 404 (error-message side-channel is audited in W6). **Status: Closed (pending W6 validation).**

**I — Identifiability:** `§ Mental model` contains the studio's interpretation of the client as a person — their anxieties, decision-making patterns, communication style. If this section were ever served to the client themselves, it would identify them as the subject of an unflattering model. **Mitigation:** Current RLS does not distinguish within-file sections; `client` role has full `canvas_files` read access for their project. **Proposed:** Audience-tag `## Mental model` as `[owner, co_owner, collaborator]` per W4. **Status: Open → `add-document-zone-permissions`.**

**N — Non-repudiation:** Mirror writes carry the `platform_owner` identity (Muse acts as the platform). The client cannot deny that their Mirror was constructed because the tool-use log (`canvas_tool_uses`) records every `write_mirror_field` invocation. This is acceptable: the artefact is the studio's work product, not the client's. **Status: Closed (accepted).**

**D — Detectability:** The existence of a `perception.md` for a given slug reveals that an engagement is active for that client. Only canvas members can detect this (RLS-gated). For invited collaborators, knowledge of the engagement is expected. **Status: Closed (accepted).**

**DD — Disclosure of information:** An `external_collaborator` assigned to a specific task in one canvas should not read the full `perception.md`, especially `§ Mental model` and `forbidden_patterns`. Currently the `canvas_files` read policy does not distinguish sections. **Proposed:** Audience-tag sensitive sections per W4 mapping. **Status: Open → `add-document-zone-permissions`.**

**U — Unawareness:** The client is not informed that a perception.md file describing their mental model is being maintained. Under GDPR Art. 14 (data not collected directly from subject), the studio must inform the client. **Proposed:** Add Mirror disclosure step to client onboarding. **Status: Open → GDPR gap (LP-MR-U-1).**

**NC — Non-compliance:** `perception.md` is constructed about the client without Art. 14 disclosure. This is a direct GDPR Art. 14 non-compliance. **Proposed:** Same as U mitigation above. **Status: Open → GDPR gap (LP-MR-NC-1).**

**Summary for CF-MR:** 2 threats closed by existing RLS + audit trail; 2 open and feeding `add-document-zone-permissions`; 2 open and feeding GDPR gap-ticket queue. No Critical-with-zero-mitigation threats; PRO NOT triggered.
