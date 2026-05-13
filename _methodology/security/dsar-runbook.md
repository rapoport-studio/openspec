---
title: DSAR runbook (Data Subject Access Request)
status: draft (v1 — batch-1 pass; end-to-end timing run pending)
owner: pavel
audience: [pavel, legal, ai]
tldr: Operational procedure for GDPR Art. 15 subject access request fulfilment.
      Target: one subject's complete data export in <60 minutes from request to deliverable.
      Produced as Phase 2 / W2 deliverable of the validate-personal-data-security change.
---

# DSAR runbook (Data Subject Access Request)

> **Legal basis:** GDPR Art. 15 (right of access); Moldova Law 133/2011 Art. 14.
> **Response deadline:** 30 calendar days from receipt; extendable once by 60 days with notification.
> **Operational target:** First draft deliverable within 60 minutes of starting the extraction procedure.
> **Companion artefacts:** [`gdpr-md-compliance-matrix.md`](gdpr-md-compliance-matrix.md) §§ Art. 15, Art. 17, Art. 20; [`breach-notification-runbook.md`](breach-notification-runbook.md).
> **Note on timing run:** The § 7 time budget section requires one end-to-end rehearsal on a synthetic subject. ⚠️ **Operator-blocked** — requires Supabase access. The runbook structure is complete; timing data will be filled in after the first rehearsal.

---

## § 1 Receipt + identity verification

### Receiving the request

Requests arrive via one of:
- Email to `pavel@rapoport.studio`
- Telegram DM to the studio account
- Written letter (rare; note receipt date for 30-day clock)

**On receipt:**
1. Log the request in a DSAR log entry (private; not in this repo). Fields: requester name, email, date received, nature of request (access / erasure / portability / all), deadline (received + 30 days).
2. Send an acknowledgement reply within 2 business days confirming the request has been received and the deadline.

### Identity verification

Verify the requester is who they claim to be before any data is disclosed.

| Requester type | Verification method |
|---|---|
| **Direct subject** (requester is a known `auth.users` email) | Confirm email matches `auth.users.email`; optionally send a magic-link challenge to confirm control of the mailbox |
| **Third-party representative** (lawyer, parent, guardian) | Require a signed authorisation letter from the subject naming the representative |
| **Third-party named subject** (requester claims to be named in someone else's data — e.g., "Anna Pavlova mentioned in a canvas message") | Verify claimed identity with at least two factors (name + email or name + phone); scope the search to text-content fields only (§ 3 query set B) |

**If identity cannot be verified:** Decline with a written explanation. Do not ask for more information than necessary (GDPR Art. 12(6) allows requesting "only" necessary additional information).

---

## § 2 Scope

For a **direct subject** (someone whose email is in `auth.users`):

| Table | Columns extracted | Join condition |
|---|---|---|
| `auth.users` | `id`, `email`, `created_at`, `last_sign_in_at`, `email_confirmed_at` | `id = $subject_id` |
| `canvas_invitees` | `canvas_id`, `role`, `created_at`, `invited_by` | `user_id = $subject_id` |
| `canvas_messages` | `canvas_id`, `content`, `created_at`, `message_type` | `user_id = $subject_id` |
| `canvas_tool_uses` | `canvas_id`, `tool_name`, `created_at`, `arguments` (redacted per § 5) | `user_id = $subject_id` |
| `project_members` | `project_id`, `role`, `created_at` | `user_id = $subject_id` |
| `inbox` | `name`, `email`, `message`, `ip_address`, `created_at` | `email = $subject_email` |
| `specialist_applications` | `name`, `email`, `skills`, `portfolio_url`, `message`, `status`, `created_at` | `email = $subject_email` |
| `community_members` | all non-admin columns | `email = $subject_email` |
| `linear_oauth_connections` | `created_at`, `updated_at` (token itself is NOT included in export — see § 4 credential exclusion) | `user_id = $subject_id` |
| `tg_user_bindings` | `tg_user_id` (hashed or redacted), `created_at` | `user_id = $subject_id` |
| `canvas_proposals` | `canvas_id`, `content`, `status`, `created_at` | `created_by = $subject_id` OR `tg_user_binding` linking to subject |

For Mirror files: all `canvas_files` rows where `path` contains `openspec/mirror/` for canvases the subject is a member of. Export the file content, redacted per § 5 for any third-party PII.

For **third-party named subjects** (name mentioned in someone else's data): scope is text-content fields only — see § 3 Query set B.

---

## § 3 Extraction queries

> **Execution:** Run these queries using a service-role Supabase connection (bypasses RLS to collect all data for the subject). Never run against the live DB without verifying `$subject_id` and `$subject_email` are correct — wrong subject = data breach.
>
> **Environment:** `psql "postgresql://postgres:[SUPABASE_SERVICE_ROLE_KEY]@db.[PROJECT_REF].supabase.co:5432/postgres"` or equivalent using `supabase db execute`.

### Query set A — Direct subject (by user_id and email)

```sql
-- Step 1: resolve subject
SELECT id, email, created_at, last_sign_in_at, email_confirmed_at
FROM auth.users
WHERE email = :'subject_email';
-- Capture returned id as :subject_id for subsequent queries.

-- Step 2: canvas memberships
SELECT ci.canvas_id, ci.role, ci.created_at, ci.invited_by,
       c.name AS canvas_name, c.created_at AS canvas_created_at
FROM canvas_invitees ci
JOIN canvases c ON c.id = ci.canvas_id
WHERE ci.user_id = :'subject_id';

-- Step 3: canvas messages authored by subject
SELECT cm.id, cm.canvas_id, cm.content, cm.created_at, cm.message_type
FROM canvas_messages cm
WHERE cm.user_id = :'subject_id'
ORDER BY cm.created_at;

-- Step 4: tool uses attributed to subject
SELECT ct.canvas_id, ct.tool_name, ct.created_at,
       ct.arguments
FROM canvas_tool_uses ct
WHERE ct.user_id = :'subject_id'
ORDER BY ct.created_at;

-- Step 5: project memberships
SELECT pm.project_id, pm.role, pm.created_at, p.name AS project_name
FROM project_members pm
JOIN projects p ON p.id = pm.project_id
WHERE pm.user_id = :'subject_id';

-- Step 6: inbox submissions
SELECT id, name, email, message, ip_address, created_at
FROM inbox
WHERE email = :'subject_email';

-- Step 7: specialist applications
SELECT id, name, email, skills, portfolio_url, message, status, created_at
FROM specialist_applications
WHERE email = :'subject_email';

-- Step 8: community membership
SELECT *
FROM community_members
WHERE email = :'subject_email';

-- Step 9: OAuth connections (existence + timestamps only — token excluded)
SELECT user_id, provider, created_at, updated_at
FROM linear_oauth_connections
WHERE user_id = :'subject_id';

-- Step 10: Telegram binding
SELECT tg_user_id, created_at
FROM tg_user_bindings
WHERE user_id = :'subject_id';

-- Step 11: canvas proposals
SELECT cp.canvas_id, cp.content, cp.status, cp.created_at
FROM canvas_proposals cp
WHERE cp.created_by = :'subject_id';
```

### Query set B — Third-party named subject (by name / email / phone pattern in text fields)

> Use only when a subject claims they are referenced in data they did not directly submit.

```sql
-- Search canvas messages for third-party name mentions
SELECT cm.id, cm.canvas_id,
       'canvas_messages' AS source_table,
       cm.created_at,
       left(cm.content, 200) AS content_excerpt
FROM canvas_messages cm
WHERE cm.content ILIKE :'%name_pattern%'
   OR cm.content ILIKE :'%email_pattern%'
   OR cm.content ILIKE :'%phone_pattern%'
ORDER BY cm.created_at;

-- Search inbox messages
SELECT id, 'inbox' AS source_table, created_at,
       left(message, 200) AS message_excerpt
FROM inbox
WHERE message ILIKE :'%name_pattern%'
   OR message ILIKE :'%email_pattern%';

-- Search canvas_tool_uses arguments
SELECT id, canvas_id, tool_name, 'canvas_tool_uses' AS source_table, created_at,
       left(arguments::text, 200) AS arguments_excerpt
FROM canvas_tool_uses
WHERE arguments::text ILIKE :'%name_pattern%'
   OR arguments::text ILIKE :'%email_pattern%';

-- Search specialist_applications for references to the named subject
SELECT id, 'specialist_applications' AS source_table, created_at,
       left(message, 200) AS message_excerpt
FROM specialist_applications
WHERE message ILIKE :'%name_pattern%';
```

### Query set C — Mirror file export

```sql
-- Get canvas_files for all canvases where subject is a member
SELECT cf.path, cf.content, cf.created_at, cf.updated_at
FROM canvas_files cf
JOIN canvases c ON c.id = cf.canvas_id
JOIN canvas_invitees ci ON ci.canvas_id = c.id
WHERE ci.user_id = :'subject_id'
  AND cf.path LIKE 'openspec/mirror/%'
ORDER BY cf.path;
```

---

## § 4 Mirror tarball generation

> ⚠️ **Operator-blocked gap:** `forge mirror-export <slug> --subject <email>` does not yet exist. This gap is tracked as RAP-XXX ("add `--subject` flag to `forge mirror-export`").

Until the flag is implemented, the manual procedure is:

1. From the Query set C results, collect all `canvas_files.content` rows where `path LIKE 'openspec/mirror/%'`.
2. Write each to a local temp directory mirroring the path structure.
3. Apply the redaction pass (§ 5) to each file.
4. Package into a tarball: `tar -czf dsar-<subject-email>-<date>.tar.gz <temp-dir>/`.
5. Deliver via § 6.

---

## § 5 Redaction pass

Before delivering any data export, apply a redaction pass to remove or hash PII of third parties referenced in the subject's data.

### Detection patterns

```
# Email-shaped references
regex: \b[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}\b

# Phone-shaped references (E.164 + common formats)
regex: (\+?[0-9]{1,3}[\s\-.]?)?(\(?[0-9]{2,4}\)?[\s\-.]?){2,4}[0-9]{4,}

# Name-shaped references adjacent to action verbs (heuristic; manual review required)
# Flag sentences containing: "call", "email", "contact", "speak to", "ask", "meet"
# followed by a capitalised word sequence
```

### Redaction rules

| Data type | Redaction method |
|---|---|
| Third-party email in free text | Replace with `[email redacted]` |
| Third-party phone in free text | Replace with `[phone redacted]` |
| Third-party name where context makes it PII | Replace with `[name redacted]` — only where the name is combined with identifying context (role, company, contact details) |
| OAuth access token | Exclude entirely from export; include only `{ "provider": "linear", "created_at": "..." }` |
| Telegram `tg_user_id` | Hash with SHA-256 before export: `sha256("tg:" || tg_user_id)` |
| IP address | Include if subject is the IP owner; redact/hash for third-party IPs in multi-user shared-IP scenarios (not currently detectable without ISP lookup) |

### Manual review step

After automated redaction, a human (platform_owner) must review the export for:
- Obvious missed third-party references
- Business-confidential information of a third party (e.g., a client's undisclosed financials mentioned in a canvas message by the studio)

---

## § 6 Delivery

### Channel options

| Method | Use case | TTL |
|---|---|---|
| Signed S3/Supabase Storage URL | Default for most requests; no email size limit | 7 days |
| PGP-encrypted email | If subject specifically requests encrypted delivery | One-time send |
| Physical medium | Only if subject is legally entitled and cannot receive digital | n/a |

### Signed URL generation (Supabase Storage)

```bash
# Upload the tarball to a private Supabase bucket
supabase storage cp dsar-<email>-<date>.tar.gz \
  ss:///dsar-exports/dsar-<email>-<date>.tar.gz

# Generate a signed URL valid for 7 days (604800 seconds)
supabase storage sign \
  dsar-exports/dsar-<email>-<date>.tar.gz \
  --expires-in 604800
```

### Covering email template

```
Subject: Your data export from Rapoport Studio — [REQUEST DATE]

Dear [NAME],

Please find your data export at the following secure link (valid for 7 days):
[SIGNED_URL]

The export includes all personal data held about you as of [EXTRACTION DATE].
It does not include data about third parties referenced in your conversations,
which has been redacted to protect their privacy.

If you have questions about this export or wish to exercise additional rights
(rectification, erasure, restriction), please reply to this email.

Reference: DSAR-[YYYYMMDD]-[SEQUENCE]

Pavel Rapoport
Rapoport Studio SRL
Data controller: Rapoport Studio SRL, Chisinau, Republic of Moldova
```

### Delete export after delivery

After the subject confirms receipt (or after 14 days with no response), delete the export from Supabase Storage. Log the deletion in the DSAR log.

---

## § 7 Time budget

> **Operational target:** 60 minutes from starting the extraction procedure to a deliverable URL sent to the subject.
> **Legal deadline:** 30 calendar days from receipt of request.

| Step | Target duration | Notes |
|---|---|---|
| Identity verification | 5 min | Email match + optional magic-link challenge |
| Query set A execution | 10 min | 11 queries; serial execution acceptable |
| Query set C (Mirror files) | 5 min | Depends on number of canvases |
| Redaction pass (automated) | 5 min | Regex patterns on exported content |
| Manual review | 20 min | Required step; hard to shortcut |
| Tarball packaging + upload | 5 min | CLI commands |
| Signed URL + covering email | 10 min | Manual drafting from template |
| **Total** | **~60 min** | **First rehearsal will calibrate this** |

> ⚠️ **Operator-blocked:** First end-to-end timing run on a synthetic subject has not been completed. This section will be updated with actual timings after the first rehearsal against the preview Supabase project. Time budget is a design target, not a measured result.

### Known slow paths

1. **Third-party named subject requests (Query set B)** — LIKE searches on `canvas_messages.content` may be slow without a full-text index. Add `GIN` index on `canvas_messages.content` if DSAR volume increases.
2. **Mirror file redaction** — manual review step is the bottleneck. `forge mirror-export --subject` flag (RAP-XXX gap) will automate this.
3. **n8n Telegram data** — currently no automated extraction path for Telegram messages retained in n8n. This is a runbook gap: n8n logs must be queried manually via the n8n admin panel. Resolution pending n8n DPA + API investigation.

---

## § 8 Erasure procedure (Art. 17)

When a subject requests erasure ("right to be forgotten") in addition to or instead of access:

```sql
-- Erasure step 1: soft-delete from application tables
DELETE FROM canvas_messages WHERE user_id = :'subject_id';
DELETE FROM canvas_tool_uses WHERE user_id = :'subject_id';
DELETE FROM canvas_invitees WHERE user_id = :'subject_id';
DELETE FROM project_members WHERE user_id = :'subject_id';
DELETE FROM canvas_proposals WHERE created_by = :'subject_id';
DELETE FROM linear_oauth_connections WHERE user_id = :'subject_id';
DELETE FROM tg_user_bindings WHERE user_id = :'subject_id';
DELETE FROM inbox WHERE email = :'subject_email';
DELETE FROM specialist_applications WHERE email = :'subject_email';
DELETE FROM community_members WHERE email = :'subject_email';

-- Erasure step 2: delete auth.users (triggers CASCADE)
-- WARNING: this is irreversible. Confirm identity one more time before executing.
DELETE FROM auth.users WHERE id = :'subject_id';
```

**Filesystem Mirror files:** Manually delete `openspec/mirror/<project-slug>/perception.md` for all projects where the subject was the primary client. Commit the deletion to git. This is a manual step until `forge mirror-export --subject --erase` flag exists.

**n8n Telegram data:** Manually delete via n8n admin panel. Document the deletion in the DSAR log.

**Backup copies:** Supabase automated backups will retain the deleted data for the backup retention period (typically 7–30 days depending on plan). This is acceptable under GDPR as long as backups are not restored and the data is excluded from any future restore. Document this in the DSAR log response letter.

---

*Authored: 2026-05-13. Batch-1 pass. End-to-end timing run operator-blocked (requires Supabase preview access). All SQL templates are for reference — verify against live schema before first execution.*
