---
title: Breach notification runbook
status: draft (v1 — batch-1 pass; template letters pending counsel review; synthetic rehearsal pending)
owner: pavel
audience: [pavel, legal, ai]
tldr: Operational procedure for security breach detection, containment, assessment, and
      72-hour regulator notification (GDPR Art. 33) + subject notification (GDPR Art. 34).
      Produced as Phase 2 / W2 deliverable of the validate-personal-data-security change.
---

# Breach notification runbook

> **Legal basis:** GDPR Art. 33 (72h regulator notification); GDPR Art. 34 (subject notification for high-risk breaches); Moldova Law 133/2011 Art. 28.
> **Regulator:** ANCD — Agenția Națională pentru Protecția Datelor cu Caracter Personal.
> **Contact:** `notificare@datepersonale.md` *(verify currency before use — see § 5)*.
> **Companion artefacts:** [`gdpr-md-compliance-matrix.md`](gdpr-md-compliance-matrix.md) §§ Art. 33, Art. 34; [`dsar-runbook.md`](dsar-runbook.md) for erasure after breach.
> **⚠️ Note on template letters:** § 5 and § 6 template letters require counsel (i-avocat) review before first use. Currently marked as DRAFT. Phase 2 close gate requires legal sign-off.
> **⚠️ Synthetic rehearsal:** A rehearsal run has not been completed. D-VS-4 note: 4-hour containment may not be realistic without on-call alerting; 12-hour is the realistic target until on-call exists.

---

## § 1 Detection

A breach is any confirmed or suspected unauthorised access, disclosure, alteration, or destruction of personal data.

### Detection sources

| Source | Signal | Action |
|---|---|---|
| Supabase logs alert | Unusual query volume, failed auth attempts, abnormal data export size | Investigate immediately |
| Cloudflare WAF alert | SQL injection attempt succeeding, unusual rate from single IP on auth endpoints | Investigate immediately |
| Manual report | Colleague or user reports unexpected data access or account behaviour | Treat as potential breach; begin assessment (§ 4) |
| User complaint | User reports receiving data that is not theirs, or reports their data appeared somewhere unexpected | Treat as confirmed breach; begin containment (§ 3) |
| Third-party disclosure | Another organisation or researcher reports finding studio data in the wild | Treat as confirmed breach; begin containment immediately |

### Severity classification (initial)

Classify within **1 hour** of detection:

| Severity | Definition | Example |
|---|---|---|
| **Critical** | Data of > 1 subject exposed externally; or credentials (Supabase service-role key, Anthropic API key) confirmed leaked | Canvas messages of 3 clients found in a public paste bin |
| **Major** | Data of exactly 1 subject exposed externally; or credentials suspected leaked | One user's inbox submission found in a competitor's hands |
| **Minor** | Data exposed within studio team beyond authorisation (e.g., a tool produced a cross-canvas result that was not disclosed externally) | External collaborator saw a canvas they were not invited to, but did not copy data |
| **Info** | Near-miss: vulnerability found but not exploited; no data exposed | Penetration tester finds an RLS gap before any user exploits it |

> **D-VS-4 note:** The 4-hour containment target (§ 3) assumes 24/7 monitoring. Until an on-call rotation exists, the realistic target for initial containment is **12 hours from detection**. Document this gap in § 7 post-mortem output for any breach that triggers GDPR Art. 33.

---

## § 2 Severity classification

Full severity decision tree (use after initial classification in § 1):

```
Is personal data of one or more data subjects confirmed or suspected to have been:
  - Accessed by an unauthorised party? → YES → assess scope below
  - Disclosed externally (published, sent to wrong recipient, etc.)? → YES → Critical or Major
  - Altered without authorisation? → YES → assess scope below
  - Destroyed or made unavailable? → YES → assess scope below

Scope:
  - > 1 subject affected externally? → Critical
  - 1 subject affected externally? → Major
  - Affected internally only (no external disclosure)? → Minor
  - No data affected; vulnerability only? → Info

Credentials:
  - Supabase service-role key, JWT secret, or Anthropic API key confirmed leaked? → Critical
  - Same credentials suspected leaked? → Major (treat as Critical until confirmed otherwise)
```

---

## § 3 Containment (target: within 4 hours; realistic: 12 hours)

Execute in order. Document each step with timestamp.

### 3.1 Revoke compromised access

```bash
# If Supabase service-role key is suspected compromised:
# 1. Go to Supabase Dashboard → Settings → API → Rotate service_role key
# 2. Update Infisical with new key:
#    infisical secrets set SUPABASE_SERVICE_ROLE_KEY=<new-key>
# 3. Redeploy Cloudflare Workers (picks up new secret from Infisical):
#    cd apps/studio && wrangler deploy

# If Anthropic API key is suspected compromised:
# 1. Go to console.anthropic.com → API Keys → Revoke compromised key
# 2. Generate new key
# 3. Update Infisical:
#    infisical secrets set ANTHROPIC_API_KEY=<new-key>
# 4. Redeploy Workers

# If a user's session is compromised:
# Revoke their Supabase session from the Auth dashboard
# User will be forced to re-authenticate via magic link
```

### 3.2 Stop the leak

- If an endpoint is actively leaking data: disable the Cloudflare Worker route temporarily (`wrangler routes delete` or add a maintenance-mode Worker).
- If a Supabase RLS policy is the issue: use the Dashboard → SQL Editor to patch the policy immediately (document the patch; formal migration follows after containment).
- If the n8n Telegram webhook is involved: disable the n8n workflow temporarily.

### 3.3 Preserve evidence

Before modifying any systems further:
- Export Supabase logs for the incident period (Dashboard → Logs).
- Export Cloudflare Worker logs (Cloudflare Dashboard → Workers → Logs).
- Screenshot relevant alerts or anomalies.
- Record all containment steps taken with timestamps.

---

## § 4 Assessment (target: within 12 hours of detection)

### 4.1 Full scope determination

| Question | How to answer |
|---|---|
| Which tables were accessed / affected? | Review Supabase query logs; look for SELECT/INSERT/UPDATE/DELETE on `canvas_messages`, `canvas_files`, `canvas_invitees`, `inbox`, `specialist_applications`, etc. |
| Which rows were affected? | Run extraction queries from DSAR runbook § 3 for each identified subject |
| Was the data copied externally? | Check Cloudflare Workers logs for large response bodies; check if any API responses were captured by the attacker |
| How long did the breach persist? | Identify the earliest anomalous log entry |
| Is the breach ongoing? | Confirm after containment (§ 3) that no further anomalous access is occurring |

### 4.2 Affected subjects enumeration

For each affected row: identify the `auth.users.email` for direct subjects; document any third-party named subjects found in the affected content.

```sql
-- Example: find subjects affected by a breach of canvas_messages
SELECT DISTINCT u.email, u.id
FROM canvas_messages cm
JOIN auth.users u ON u.id = cm.user_id
WHERE cm.canvas_id = :'affected_canvas_id'
  AND cm.created_at BETWEEN :'breach_start' AND :'breach_end';
```

### 4.3 Root cause analysis

Document:
1. How did the attacker gain access? (Credential theft / RLS bypass / application vulnerability / insider)
2. Which component failed? (Supabase RLS / server action / n8n webhook / Cloudflare Worker)
3. Was this a known gap from the LINDDUN threat model or adversarial RLS audit?

---

## § 5 Regulator notification (within 72 hours of becoming aware)

**Recipient:** ANCD — `notificare@datepersonale.md`  
**Deadline:** 72 hours from when the controller (Pavel) became aware of the breach.  
**Form:** Email with the notification letter below.

> ⚠️ **Verify currency of ANCD contact before sending.** Check `datepersonale.md` for current notification instructions. The email above was current as of 2026-05-13.

### Template letter (DRAFT — requires i-avocat review before first use)

```
Subject: Notification of personal data breach — Rapoport Studio SRL — [DATE]

To: Agenția Națională pentru Protecția Datelor cu Caracter Personal
    notificare@datepersonale.md

---

NOTIFICATION OF PERSONAL DATA SECURITY BREACH
pursuant to GDPR Art. 33 and Moldova Law 133/2011 Art. 28

Controller:
  Rapoport Studio SRL
  [Legal address — Chisinau, Republic of Moldova]
  Contact: Pavel Rapoport, pavel@rapoport.studio

Date and time breach discovered: [TIMESTAMP]
Date of this notification: [DATE]

1. Nature of the breach
   [Describe: what happened, how it was discovered, which systems were involved]

2. Categories of personal data affected
   [e.g., Identity data (name, email), communication data (canvas messages),
   professional data (specialist applications)]

3. Approximate number of data subjects affected
   [N subjects; or "unknown — assessment ongoing" if not yet determined]

4. Likely consequences
   [e.g., "Unauthorised access to canvas messages may enable the recipient to
   understand the client's internal business strategy"]

5. Measures taken or proposed
   a) Containment: [describe actions taken per § 3]
   b) Remediation: [describe proposed fix]
   c) Preventive: [describe how recurrence will be prevented]

6. Contact for further information
   Pavel Rapoport
   pavel@rapoport.studio
   [Phone if requested]

---
[Signature]
Pavel Rapoport
Rapoport Studio SRL
[Date]
```

### What qualifies as a notifiable breach

Under GDPR Art. 33(1) and MD Law 133/2011, notification is required unless "the personal data breach is unlikely to result in a risk to the rights and freedoms of natural persons." In practice:

- **Critical and Major severity:** Always notify ANCD within 72 hours.
- **Minor severity:** Assess risk. If data exposed only internally with no external leak risk, notification may not be required — but document the assessment.
- **Info severity:** No notification required; document internally.

---

## § 6 Subject notification (for Critical / Major breaches)

Subject notification is required under GDPR Art. 34 when the breach is "likely to result in a high risk to the rights and freedoms of natural persons."

**Timing:** Notification "without undue delay" — in practice, within 72 hours of ANCD notification, i.e., within 144 hours of discovery.

### Template letter (DRAFT — requires i-avocat review before first use)

```
Subject: Important security notice regarding your data at Rapoport Studio

Dear [NAME],

We are writing to inform you of a security incident that may have affected
your personal data held by Rapoport Studio SRL.

What happened:
[Plain-language description of the breach]

What data was involved:
[Specify: e.g., "Your email address and project communications stored in our system"]

What we have done:
[Describe containment actions taken]

What you can do:
[If applicable: change passwords elsewhere if same credentials used, be alert
to phishing emails using your name, etc.]

Your rights:
You have the right to contact us for more information, to request access to your
data, to request deletion, or to lodge a complaint with the ANCD:
  datepersonale.md | notificare@datepersonale.md

We apologise for this incident and take full responsibility for the security
of your data.

Pavel Rapoport
Rapoport Studio SRL
[Date]
Reference: BREACH-[YYYYMMDD]-[SEQUENCE]
```

---

## § 7 Post-mortem

Every Critical or Major breach requires a post-mortem document published within 14 days.

**Location:** `_methodology/security/post-mortems/<YYYY-MM-DD>-<slug>.md`

### Post-mortem template structure

```markdown
# Post-mortem: [breach description] — [YYYY-MM-DD]

## Summary
[2-3 sentence plain-language summary of what happened and its impact]

## Timeline
| Time | Event |
|---|---|
| [T+0] | Breach detected |
| [T+Xh] | Containment started |
| [T+Xh] | Containment completed |
| [T+72h] | ANCD notification sent (if applicable) |
| [T+Xh] | Subject notification sent (if applicable) |

## Root cause
[Technical root cause + contributing factors]

## Impact
- Subjects affected: N
- Data categories: [list]
- Duration of exposure: [time window]
- External exposure: [yes/no; details]

## What went well
[What the response got right]

## What went wrong
[What failed; process gaps; detection gaps]

## Action items
| Action | Owner | Due date |
|---|---|---|
| [Fix] | pavel | [date] |

## LINDDUN / audit artefact updates required
[List any canvas-permission-linddun.md threats or canvas-rls-adversarial-audit-2026.md
scenarios that should be updated based on this incident]
```

---

## § 8 Synthetic rehearsal record

> ⚠️ **Operator-blocked:** Synthetic rehearsal has not been completed. Phase 2 close gate requires one full rehearsal. The scenario: "Anthropic API key found in a Cloudflare Worker log line by Pavel during routine log review."

**Rehearsal scenario:** Pavel notices during Cloudflare log review that an Anthropic API key appears in a Worker response body (a developer accidentally logged it during debugging). This constitutes a potential credential breach.

**Planned rehearsal steps:**
1. Classify severity: Major (credential suspected leaked; no confirmed external access yet)
2. Containment: Rotate Anthropic API key (§ 3.1); deploy updated Worker; preserve evidence
3. Assessment: Review CF logs for any external access to the leaked key; enumerate affected prompts (if any)
4. ANCD notification assessment: likely not required (credentials only, no subject data confirmed exposed) — document assessment
5. Update runbook based on timing and friction points

**Expected friction points (hypothesis):**
- Locating the exact log line that exposed the key
- Confirming no external access during the exposure window (log retention period)
- 4-hour containment target: achievable; 12-hour more realistic without alerting automation

*Rehearsal to be conducted after this runbook merges. Timings and findings will be captured here.*

---

*Authored: 2026-05-13. Batch-1 pass. Template letters require counsel review (operator-blocked). Synthetic rehearsal operator-blocked. Phase 2 close gate: artefact landed ✓; counsel review pending; rehearsal pending.*
