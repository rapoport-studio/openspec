---
title: Canvas RLS adversarial audit 2026
status: draft (v1 — harness landed; first run operator-blocked pending Supabase preview credentials)
owner: pavel
audience: [security-specialist, pavel, ai]
tldr: Reusable adversarial RLS audit harness for the studio platform. Documents the
      test framework, 15-scenario test matrix, and re-run cadence. Produced as
      Phase 3 / W5 deliverable of the validate-personal-data-security change.
      First run results will populate § 3 Findings when Supabase preview credentials
      are available.
---

# Canvas RLS adversarial audit 2026

> **Companion to:** [`openspec/archive/2026-05-13-validate-personal-data-security/`](../../archive/2026-05-13-validate-personal-data-security/)
> **Harness location:** [`tools/security/rls-redteam.sh`](../../../tools/security/rls-redteam.sh)
> **Scenarios location:** [`tools/security/scenarios/`](../../../tools/security/scenarios/)
> **First run status:** ⛔ **operator-blocked** — requires Supabase preview credentials (D-VS-2 default: preview for routine; live monthly with explicit gate)

---

## § 1 Test framework

### 1.1 Overview

The harness at `tools/security/rls-redteam.sh` synthesises PostgreSQL JWTs for each canvas role, executes a suite of SQL scenarios against the Supabase project, and diffs actual output against expected `.json` files. The goal is to catch cross-canvas disclosure, privilege escalation, and confused-deputy vulnerabilities in RLS policies before they reach production.

### 1.2 Roles under test

| Role | JWT claims | Test priority |
|---|---|---|
| `platform_owner` | `app_metadata.is_platform_owner = true` | Baseline — should see everything |
| `owner` | `app_metadata.canvas_role = owner` + canvas membership | High — broadest per-canvas access |
| `co_owner` | `app_metadata.canvas_role = co_owner` | High |
| `collaborator` | `app_metadata.canvas_role = collaborator` | High |
| `external_collaborator` | `app_metadata.canvas_role = external_collaborator` | **Highest** — new trust boundary; cohort 1 of `add-specialist-onboarding` |
| `viewer` | `app_metadata.canvas_role = viewer` | High |
| `client` | `app_metadata.canvas_role = client` | High |
| `stranger` | authenticated user with no canvas membership | High — should see nothing in canvas tables |
| `anon` | unauthenticated (no JWT) | Medium — PostgREST anon key tests |

### 1.3 Harness execution model

```
rls-redteam.sh [--env preview|live] [--scenario <id>] [--role <role>]

  --env preview    Use SUPABASE_URL_PREVIEW + SUPABASE_SERVICE_ROLE_KEY_PREVIEW
  --env live       Use SUPABASE_URL + SUPABASE_SERVICE_ROLE_KEY (monthly gate only)
  --scenario <id>  Run single scenario (e.g. 01-confused-deputy)
  --role <role>    Run all scenarios for a single role
  (no args)        Run full suite against --env preview
```

For each role × scenario combination:
1. Mint a synthetic JWT using the service-role key (impersonation via `X-Supabase-Auth` header or direct Postgres `SET LOCAL role` + `SET LOCAL request.jwt.claims`)
2. Execute the scenario SQL via `psql` with `PGRST_JWT_SECRET`-signed token
3. Capture: row count, error code, error message, latency (ms)
4. Diff against `scenarios/<scenario>.expected.json`
5. Emit: PASS / FAIL / WARN per role

Output formats: `--format table` (default), `--format junit` (CI integration), `--format json` (machine-readable).

### 1.4 JWT minting approach

```bash
# Using supabase CLI for preview environment
supabase functions invoke mint-test-jwt \
  --project-ref $SUPABASE_PROJECT_REF_PREVIEW \
  --body '{"role": "external_collaborator", "canvas_id": "...", "user_id": "..."}'
```

Fallback (direct psql with service-role bypass):
```sql
-- Within psql session authenticated with service-role key:
SET LOCAL role = authenticated;
SET LOCAL "request.jwt.claims" = '{"sub": "<synthetic-uuid>", "role": "authenticated",
  "app_metadata": {"canvas_role": "external_collaborator"}}';
-- Then execute scenario SQL
```

### 1.5 Test data setup

Each harness run uses a **fixed synthetic dataset** seeded before the suite begins:

| Synthetic entity | Value |
|---|---|
| Canvas A (`platform_owner`-owned) | UUID `00000000-0000-0000-0000-000000000001` |
| Canvas B (foreign project, same Supabase instance) | UUID `00000000-0000-0000-0000-000000000002` |
| `external_collaborator` member of Canvas A | UUID `00000000-0000-0000-0001-000000000001` |
| `viewer` member of Canvas A | UUID `00000000-0000-0000-0001-000000000002` |
| `stranger` (no canvas membership) | UUID `00000000-0000-0000-0001-000000000003` |
| Canvas B messages (should be invisible to Canvas A members) | seeded rows |
| `platform_owner` user | UUID `00000000-0000-0000-0001-000000000000` |

Seed script: `tools/security/seed-test-data.sql` (committed alongside scenarios).

### 1.6 Known limitations

- Tests only Supabase RLS layer; does not catch logic bugs in server actions that call Supabase with the service-role key bypassing RLS entirely.
- JWT synthesis requires service-role key; test environment must be isolated (preview, not live).
- `anon` role test requires PostgREST `anon` key; not the same as `service_role` JWT tests.
- Latency measurements are indicative only; run from CI runner proximity to Supabase region.

---

## § 2 Test matrix

15 scenarios authored in batch-1. Remaining 10 scenarios will be added after W1 LINDDUN review (per OQ-1 in `canvas-permission-linddun.md § 5`).

| ID | Scenario name | Table(s) | Attack description | Roles expected to fail | Severity if RLS gap |
|---|---|---|---|---|---|
| 01 | confused-deputy | `canvas_messages` | Canvas A member reads Canvas B messages by referencing Canvas B UUID directly | `external_collaborator`, `viewer`, `client`, `stranger` | Critical |
| 02 | slug-enumeration | `canvas_files` | Authenticated stranger enumerates Mirror slugs by scanning `canvas_files` paths | `stranger`, `anon` | High |
| 03 | role-escalation-collab | `canvas_invitees` | `collaborator` attempts to insert a new `owner`-role invite | `collaborator`, `external_collaborator`, `viewer`, `client` | Critical |
| 04 | external-collab-canvas-files | `canvas_files` | `external_collaborator` reads `perception.md` (Mental model section); no zone-permission gate yet | `external_collaborator` | High |
| 05 | viewer-tool-use-write | `canvas_tool_uses` | `viewer` attempts to insert a tool-use record on behalf of another user | `viewer`, `client`, `stranger` | Medium |
| 06 | stranger-inbox-read | `inbox` | Authenticated stranger reads inbox leads | `stranger`, `collaborator`, `external_collaborator`, `viewer`, `client` | Critical |
| 07 | stranger-specialist-apps | `specialist_applications` | Authenticated stranger reads specialist application records | `stranger`, `collaborator`, `external_collaborator`, `viewer`, `client` | Critical |
| 08 | oauth-token-read | `linear_oauth_connections` | `external_collaborator` reads another user's Linear OAuth access token | `external_collaborator`, `viewer`, `client`, `stranger` | Critical |
| 09 | tg-binding-read | `tg_user_bindings` | `collaborator` reads another user's Telegram binding | `collaborator`, `external_collaborator`, `viewer`, `client`, `stranger` | High |
| 10 | project-connection-config | `project_connections` | `viewer` reads raw `config` JSON including API credentials | `viewer`, `client`, `stranger`, `external_collaborator` | Critical |
| 11 | canvas-delete | `canvas_messages` | `external_collaborator` deletes messages in Canvas A | `external_collaborator`, `viewer`, `client`, `stranger` | High |
| 12 | foreign-canvas-invitees | `canvas_invitees` | `owner` of Canvas A reads membership list of Canvas B | `owner`, `co_owner`, `collaborator`, `external_collaborator`, `viewer`, `client`, `stranger` | High |
| 13 | canvas-files-update | `canvas_files` | `viewer` updates a `canvas_files` row in their canvas | `viewer`, `client`, `external_collaborator`, `stranger` | High |
| 14 | project-member-escalation | `project_members` | `collaborator` updates their own role to `owner` | `collaborator`, `external_collaborator`, `viewer`, `client`, `stranger` | Critical |
| 15 | anon-canvas-read | `canvas_messages` | Unauthenticated request (anon key) reads any canvas messages | `anon` | Critical |

**Scenario count note:** batch-1 authors 15 scenarios. Design.md § 6.1 calls for ~25 after W1 LINDDUN completes. Remaining 10 will be added when OQ-1 is resolved.

---

## § 3 Findings

> ⛔ **operator-blocked** — first run requires Supabase preview credentials (D-VS-2 default). This section will be populated after the first run.

**Placeholder structure (to be filled after first run):**

| ID | Scenario | Role | Expected | Actual | Classification | Remediation |
|---|---|---|---|---|---|---|
| F-001 | _(first run)_ | _(first run)_ | _(first run)_ | _(first run)_ | _(first run)_ | _(first run)_ |

**Classification key:**
- **Critical** — cross-canvas disclosure or privilege escalation; block merge
- **Major** — partial disclosure within authorised canvas; address before next release
- **Minor** — information leakage not directly exploitable; track-but-not-block
- **Info** — unexpected but harmless behaviour; document and accept

**Success criterion:** Zero critical / zero major findings on first run against preview Supabase. Minor findings become Linear sub-issues.

---

## § 4 Re-run cadence

| Trigger | Environment | Who approves |
|---|---|---|
| Every PR touching `supabase/migrations/**` or any file matching `**/rls/**` | Preview | Automatic (CI) |
| Monthly on the 1st | Preview | Automatic (cron) |
| Before `add-specialist-onboarding` cohort 1 launch | Live (with explicit gate) | Platform owner |
| Before `add-document-zone-permissions` merge | Preview | Platform owner |
| Ad hoc when new role or table added | Preview | Platform owner |

**CI integration:** `.github/workflows/rls-audit.yml` is tracked as a follow-up (out-of-scope for Phase 3 close; see tasks.md Phase 3 CI integration item).

**Run summary attachment:** PR description must include the harness summary table output (copy-paste from `--format table`). This is the `db/conventions.md` amendment that lands at archive.

---

## § 5 Known limitations

1. **Server-action bypass:** Server actions that use the Supabase service-role key bypass RLS entirely. The harness does not test those code paths. Manual review of `apps/studio/src/app/actions/**` is required (conducted in W6 side-channel review).
2. **JWT claim accuracy:** Synthetic JWTs may not perfectly mirror production claim shapes if `app_metadata` structure evolves. Harness must be updated when claim shapes change.
3. **Preview drift:** Preview database may lag behind production schema if migrations are not applied to preview. Run `supabase db push --project-ref $PREVIEW_REF` before each suite run.
4. **Row-level vs column-level:** RLS policies operate at row level. Column-level exposure within an authorised row (e.g., `project_connections.config` fields) is not tested by row-count checks alone; scenario 10 requires manual inspection of returned JSON shape.
5. **Concurrency:** Harness runs scenarios serially; concurrent-write race conditions in RLS are not tested by this harness.
