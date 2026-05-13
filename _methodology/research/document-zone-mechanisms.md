---
title: Document zone mechanisms — sub-document access design
status: draft (v1 — batch-1 pass; W3 content audit uses synthetic artefacts per D-VS-1 default)
owner: pavel
audience: [architect, pavel, ai]
tldr: W3 content audit + W4 sub-document access mechanism research for the studio
      platform. Produces the per-section Mirror audience mapping that is the input
      artefact for add-document-zone-permissions Phase 0. Companion to the
      validate-personal-data-security change.
---

# Document zone mechanisms — sub-document access design

> **Companion to:** [`openspec/changes/validate-personal-data-security/`](../../changes/validate-personal-data-security/)
> **Consumes:** W1 LINDDUN output (`canvas-permission-linddun.md`) for sensitivity signals
> **Produces:** § 4 per-section Mirror mapping — the input artefact for `add-document-zone-permissions` Phase 0
> **D-VS-1 resolution:** v1 content audit uses synthetic artefacts (representative but not real engagement data). Real files may be audited with founder consent + light anonymisation in v2.

---

## § 1 Content audit

### 1.1 Sensitivity tier scale

| Tier | Definition | Example |
|---|---|---|
| **Public** | Safe for any audience including unauthenticated visitors | Mirror render of a marketing block; `## Success phrase` |
| **Internal** | Safe within the studio + the project; not for outsiders | `## Aspirational tone`; `key_screens[].name` |
| **Confidential** | Safe within a subset of project members; specific audience required | `## Mental model`; `forbidden_patterns` |
| **Restricted** | Safe within a tightly-scoped set (owner + co_owner only); legal-sensitive | NDA-protected business model notes; revenue projections |

### 1.2 Content audit table

15 representative artefacts. 5 synthetic Mirror perception.md sections, 5 OpenSpec change folder entries, 5 canvas_messages examples.

| Artefact ID | Artefact type | Content type | Sensitivity tier | Audience recommendation | Notes |
|---|---|---|---|---|---|
| CA-01 | Mirror `client_vocabulary` entry | Client-specific terminology | **Internal** | `[*]` (all roles) | Vocabulary is safe for client to confirm; useful for external_collaborator context |
| CA-02 | Mirror `## Mental model` section | Studio's interpretation of client psychology, anxieties, decision patterns | **Confidential** | `[owner, co_owner, collaborator]` | Client or external_collaborator reading this section would find it revealing and potentially offensive; founder did not consent to sharing their mental model annotation |
| CA-03 | Mirror `forbidden_patterns` | Phrases the studio should never use with this client | **Confidential** | `[owner, co_owner, collaborator]` | Sharing with external_collaborator reveals internal positioning strategy |
| CA-04 | Mirror `## Aspirational tone` | Desired emotional register for deliverables | **Internal** | `[owner, co_owner, collaborator, external_collaborator]` | Safe for external_collaborator; helps them calibrate output |
| CA-05 | Mirror `## Success phrase` | One phrase that captures project success | **Public** | `[*]` | Safe to share broadly; used in marketing |
| CA-06 | Mirror `key_screens[].name` | Screen or deliverable name | **Internal** | `[*-client]` default | Default: all roles including client; per-item override possible |
| CA-07 | Mirror `brand_anchors` | Brand positioning reference anchors | **Internal** | `[*-client]` default | Brand identity is collaborative; client can see |
| CA-08 | OpenSpec change folder: `proposal.md` | Engineering change proposal | **Internal** | `[platform_owner, collaborator]` | Not canvas data; in version control; not subject to RLS |
| CA-09 | OpenSpec change folder: `design.md` | Architectural design decision | **Internal** | `[platform_owner, collaborator]` | Not canvas data |
| CA-10 | OpenSpec change folder: `tasks.md` | Implementation checklist | **Internal** | `[platform_owner, collaborator]` | Not canvas data |
| CA-11 | `canvas_messages`: Muse coaching response | AI coaching output in canvas chat | **Confidential** | `[canvas members]` (RLS-governed) | Governed by existing canvas RLS; no sub-document access needed |
| CA-12 | `canvas_messages`: client-sent message | Client's message about their project | **Confidential** | `[canvas members]` (RLS-governed) | Same as above |
| CA-13 | `canvas_messages`: internal studio note | Studio's note not intended for client | **Confidential** | `[owner, co_owner, collaborator]` | A "not for client" message type; currently no mechanism to tag canvas_messages by audience; this is out-of-scope for v1 zone permissions (section-level, not message-level) |
| CA-14 | Mirror `out_of_scope` frontmatter field | What the project explicitly excludes | **Public** | `[*]` | Safe to share; helps align expectations |
| CA-15 | Mirror `comparable_products[]` entry | Products the studio internally considers analogous to client's | **Confidential** | `[owner, co_owner, collaborator]` | Reveals strategic positioning; not for client consumption |

### 1.3 Sensitivity histogram

| Tier | Count | % |
|---|---|---|
| Public | 3 | 20% |
| Internal | 5 | 33% |
| Confidential | 6 | 40% |
| Restricted | 1 | 7% |

**Finding:** The majority of content in Mirror files is Confidential or Internal, not Public. The primary risk surface is the studio's internal interpretation of the client (CA-02 Mental model, CA-03 forbidden_patterns, CA-15 comparable_products) — content the client did not produce and should not see unless explicitly approved by the founder.

**Implication for W4:** Variant α (audience tags at the section level) is sufficient for the current content distribution. Variant β (block-level rows) is over-engineering for v1 unless the `canvas_messages` internal-note case (CA-13) becomes a priority.

---

## § 2 Sub-document access mechanism options

### Option α — Audience tags in Mirror frontmatter / section headers

**Model:** Each Mirror frontmatter field and body section carries an `audience` tag. Tags are arrays of roles. The render pipeline and read API filter sections based on the requesting user's role.

**Data model implications:**
- No schema change to `canvas_files`
- Tags live inside the YAML/Markdown content itself
- Filter applied at read time (application layer)

**RLS implications:** None — RLS is unchanged. Filtering is post-RLS at the application layer. A service-role bypass would still expose raw content; guard server actions accordingly.

**Render implications:** Render pipeline (Forge) reads audience tags and omits sections. Unauthenticated renders serve only `[*]`-tagged sections. This is the primary motivator.

**UI implications:** Section-level access indicator in the Mirror editor (optional v1 enhancement). No structural UI change required for MVP.

**Audit-trail implications:** Section access decisions are logged in server action log (existing pattern). No additional schema needed.

**Performance implications:** Negligible. Tag evaluation is a simple array inclusion check per section at parse time.

**Cost of implementation:** Low. Primarily a change to the Forge render pipeline and Mirror read server action. Estimated: 2–3 days engineering.

### Option β — Block-level rows in a separate table

**Model:** Each logical "block" of a Mirror document is stored as a separate row in a `canvas_document_blocks` table. Each row has an `audience` column. RLS enforces access at the block level.

**Data model implications:** Major schema change. `canvas_files.content` is replaced (or supplemented) by normalised block rows. Migration of existing Mirror files required.

**RLS implications:** RLS policies on `canvas_document_blocks` per audience. This is structurally sound but adds complexity to all Mirror read paths.

**Render implications:** Render pipeline fetches only authorised blocks per requesting role. Clean but requires significant Forge refactor.

**UI implications:** Block-level editor with per-block audience selector. Non-trivial UI engineering.

**Audit-trail implications:** Row-level access is naturally auditable (existing RLS audit model).

**Performance implications:** N+1 read patterns if blocks are fetched individually; batching required. Materialised view for full-document reads.

**Cost of implementation:** High. 1–2 weeks engineering across schema, Forge, Muse, Studio UI.

**When β becomes necessary:** When `canvas_messages` internal-note case (CA-13) becomes a priority, i.e., when the studio needs to mark individual messages (not sections) as internal-only.

### Option γ — Field-level encryption

**Model:** Sensitive fields (Mental model, forbidden_patterns) are encrypted at the application layer before writing to `canvas_files`. Decryption key scoped to authorised roles.

**Data model implications:** No schema change. Encrypted blob in content field. Key management complexity.

**Security properties:** Strongest at-rest protection. Even service-role bypass cannot read without the key.

**Cost of implementation:** Very high. Key management, rotation, DSAR extraction all require re-design.

**When γ becomes necessary:** When a client contract requires data-at-rest isolation guarantees that Supabase TDE alone does not satisfy. Currently: out of scope. Filed as `add-field-level-encryption`.

### Option δ — Render-time redaction

**Model:** Full content is stored and served to authorised roles. At render time, sections above the requesting role's clearance are redacted (replaced with placeholder text). No application-layer filtering; pure render pipeline concern.

**Security properties:** Weakest. Raw content is accessible to any role that can call the Mirror read API; only the rendered output is redacted. A direct API call bypasses redaction.

**When δ is appropriate:** Public-facing renders only, where the audience is unauthenticated and the redaction is a UX choice, not a security control. Not appropriate as the primary access control mechanism.

---

## § 3 Comparative literature review

### Systems surveyed

| System | Sub-document access mechanism | Pattern |
|---|---|---|
| **Notion** | Block-level permissions (Notion pages are trees of blocks; each block inherits from page; page-level access only in free tier; block-level in enterprise) | Inheritance with enterprise override |
| **Figma** | Frame/component permissions (view access is page-level; branch permissions in enterprise) | Inheritance; no block-level in standard tier |
| **Linear** | Issue access is project-level; comment visibility tied to issue access | Inheritance |
| **Slack** | Channel-level access; thread visibility tied to channel; no sub-message access control | Inheritance |
| **Quip** | Section-level comments; no section-level access control | No sub-document access |
| **Granola** | Note sections; no section-level access control (all-or-nothing per note) | None |
| **1Password** | Item-level access via Vault + Group; no field-level in standard; field-level masking (reveal-on-demand) in enterprise | Vault/Group inheritance; masking not encryption |
| **GitHub Issues** | Issue-level access; no comment-level; private issues are all-or-nothing | Inheritance |
| **HashiCorp Vault** | Secret path hierarchy; ACL policies at path level; field-level via response wrapping | Path-based ACL; strongest model |
| **Google Docs** | Document-level sharing; no section-level; comment visibility = document visibility | Inheritance |

### Distilled patterns

1. **Inheritance (dominant):** Most systems apply access control at the document/resource level; sub-document granularity is either absent or an enterprise add-on. Sub-document access control is rare in the wild and always adds engineering complexity.

2. **Override:** Systems that do support sub-document access (Notion enterprise, HashiCorp Vault) use a path/block-level override that inherits from the parent unless explicitly overridden.

3. **Redaction vs encryption:** 1Password uses masking (redaction) for field values — the value is in the API response but hidden in the UI until explicitly revealed. HashiCorp Vault uses response wrapping (encryption). The studio's v1 need is masking/filtering, not encryption.

4. **Ephemeral access:** No surveyed system uses ephemeral decryption keys for sub-document access in standard tier.

**Takeaway for the studio:** Variant α (audience tags with application-layer filtering) closely mirrors the Notion enterprise model (section-level inheritance with override), implemented at a scale appropriate to the studio's current engineering bandwidth. It is the pragmatic choice for v1.

---

## § 4 Recommendation + per-section Mirror mapping

### 4.1 Recommended variant

**Variant α — audience tags in Mirror frontmatter and section headers.**

Rationale:
- Sufficient for the identified content distribution (§ 1.3: 40% Confidential, 33% Internal, 20% Public)
- Low implementation cost (2–3 days vs 1–2 weeks for β)
- Precedent in Notion enterprise; well-understood pattern
- No schema migration required
- Preserves existing RLS as the primary security control; audience tags are an application-layer filter
- β trigger clearly defined (CA-13 message-level case) — upgrade path documented

**When β becomes necessary:**
- If the founder needs to mark individual `canvas_messages` as "internal only" (not just Mirror sections)
- If a client contract requires cryptographically-enforced sub-document isolation (triggers γ instead)

### 4.2 Per-section Mirror mapping (input for `add-document-zone-permissions`)

This table is the direct input for `add-document-zone-permissions` Phase 0.

| Mirror field / section | Audience tag | Rationale | Notes |
|---|---|---|---|
| `client_vocabulary[].client_term` | `[*]` | Vocabulary is collaborative; safe for all | `use_in_renders` flag already gates render inclusion |
| `client_vocabulary[].canonical` | `[*]` | Canonical term is safe for all | |
| `key_screens[].name` | `[*-client]` default; per-item override | Screen names are Internal by default; client can view with explicit approval | Override mechanism needed for items approved for client sharing |
| `brand_anchors` | `[*-client]` default | Brand positioning is Internal | |
| `forbidden_patterns` | `[owner, co_owner, collaborator]` | Reveals internal positioning strategy; not for external_collaborator or client | Critical: external_collaborator hired for specific tasks should not know what to avoid saying |
| `out_of_scope` | `[*]` | Alignment artefact; safe for all | |
| `comparable_products[]` | `[owner, co_owner, collaborator]` | Strategic positioning; not for client or external_collaborator | |
| `## Mental model` | `[owner, co_owner, collaborator]` | Studio's private interpretation of client; not for client or external_collaborator | **Highest sensitivity section in Mirror** |
| `## Aspirational tone` | `[owner, co_owner, collaborator, external_collaborator]` | Tone calibration is useful for external_collaborator; safe to share | |
| `## Success phrase` | `[*]` | One-line success definition; public-safe | Often used in marketing |
| `## Worked examples` (if present) | `[owner, co_owner, collaborator]` | May contain client-specific examples; default to Internal | |
| Frontmatter `title` | `[*]` | Project name; already visible in canvas | |
| Frontmatter `status` | `[owner, co_owner, collaborator]` | Workflow status is internal | |

### 4.3 Audience tag notation

Tags are arrays of role identifiers. Special values:
- `[*]` — all authenticated users with canvas membership (does not include unauthenticated)
- `[*-client]` — all canvas members except `client` (equivalent to `[owner, co_owner, collaborator, external_collaborator, viewer]`)
- `[public]` — included in unauthenticated / public renders

Role identifiers match `redesign-canvas-permission-model § 3 Role hierarchy`:
`platform_owner`, `owner`, `co_owner`, `collaborator`, `external_collaborator`, `viewer`, `client`

### 4.4 Default access policy (when no tag is present)

For backward compatibility: **any Mirror section without an audience tag is treated as `[owner, co_owner, collaborator]`** (Internal tier). This prevents untagged content from becoming inadvertently public.

---

## § 5 Open questions for `add-document-zone-permissions`

| OQ | Question | Impact | Resolution |
|---|---|---|---|
| DZM-OQ-1 | How does the audience tag interact with the Forge render pipeline when a `[*-client]` section is included in a render served directly to the client role? | Render may expose Internal content to client if render path is not role-aware | Forge render must accept a `--role` flag to filter sections by audience |
| DZM-OQ-2 | Should `comparable_products` be configurable per-item (some products safe to show client) or always whole-field? | Per-item would require a nested tag structure | Start whole-field; add per-item override in v2 |
| DZM-OQ-3 | How is the `external_collaborator`'s specific task scope communicated to the system? (They see `Aspirational tone` but not `Mental model`; but a highly specialised collaborator might need neither.) | Over-sharing to external_collaborator is a real risk | For v1, accept `external_collaborator` sees all `[..., external_collaborator]` sections; per-collaborator overrides in v2 |
| DZM-OQ-4 | What happens to existing Mirror files that predate the audience tag system? | Migration needed | Default all untagged sections to `[owner, co_owner, collaborator]` per § 4.4 default policy |
