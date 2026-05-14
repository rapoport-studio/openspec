---
title: Threat model template (Shostack 4-Question Frame)
status: stable
owner: pavel
audience: [security-specialist, pavel, ai]
tldr: Lightweight 4-question template per capability with auth or secrets surface. Copy into capability folder when needed.
---

# Threat model template (Shostack 4-Question Frame)

> Lightweight variant of [Adam Shostack's 4-Question Frame](https://github.com/adamshostack/4QuestionFrame).
> Heavy DFD/STRIDE-per-element analysis is **optional**. The lightweight 4-question version is **mandatory** for any capability with auth, secrets, or PII surface.
>
> **How to use:** Copy this file into the relevant capability folder (e.g., `openspec/current/forge-threat-model.md`) and fill in the sections. Use the Forge capability worked example below as a reference.

---

## How to use this template

1. Copy the file. Rename to `<capability>-threat-model.md`.
2. Fill in the four sections. Each has guiding prompts + a Forge-capability worked example.
3. For capabilities with auth, RLS, or secrets surface — consult [`security-review-checklist.md`](security-review-checklist.md) in parallel.
4. "Did we do a good job?" is answered at PR review time, not during authoring.

---

## Q1 — What are we working on?

> _Identify the system: actors, trust boundaries, data flows, data classifications._

### Guiding prompts

- Who are the actors? (human users, other services, CI pipelines, AI agents)
- What data does the capability touch? (PII, auth tokens, secrets, financial data, user content)
- Where are trust boundaries? (public internet → service edge, service → DB, CI → secrets store)
- What is in scope for this threat model? What is explicitly out of scope?

### Optional: data flow sketch

```
[Actor A] ──(data: X)──▶ [Component B] ──(call: Y)──▶ [Store C]
                                  │
                          trust boundary
```

### Worked example — Forge CLI

| Element | Detail |
|---|---|
| Actors | Developer (trusted), CI runner (trusted-restricted), GitHub Actions (trusted-restricted), Forge AI agent (semi-trusted — AI output is not implicitly trusted) |
| Data | Linear issue content, repo file contents, Anthropic API requests/responses, GitHub PR metadata |
| Trust boundaries | Developer workstation → GitHub (HTTPS + PAT); CI → GitHub Actions Secrets (env injection); Forge → Anthropic API (HTTPS + bearer token); Forge → Linear API (OAuth token) |
| In scope | Forge plan/build pipeline, secret consumption, orchestrator state (`~/.forge/locks/`) |
| Out of scope | Anthropic model internals, GitHub platform security |

---

## Q2 — What can go wrong?

> _Enumerate threats. Use STRIDE-light for each trust boundary crossing, or MITRE ATLAS tactics for AI-touching surfaces._

### STRIDE-light reference (use for service-to-service boundaries)

| Letter | Threat category | Example |
|---|---|---|
| S | Spoofing | Actor pretending to be a different actor |
| T | Tampering | Unauthorised modification of data or code |
| R | Repudiation | Denying an action was taken; no audit trail |
| I | Information disclosure | Data accessible to unintended parties |
| D | Denial of service | Resource exhaustion, crash, lockout |
| E | Elevation of privilege | Actor gaining more permissions than intended |

### MITRE ATLAS tactics (use for AI-facing surfaces)

Relevant tactics for Forge-class agents: **Initial Access** (prompt injection via repo files), **Execution** (AI-generated code with unintended side-effects), **Persistence** (AI modifying its own config), **Exfiltration** (AI leaking secrets via API calls).

### Threat enumeration table

| ID | Threat | Component | STRIDE/ATLAS | Likelihood | Impact | RPN (L×I) |
|---|---|---|---|---|---|---|
| T1 | _description_ | _component_ | _category_ | 1–5 | 1–5 | _L×I_ |

> **RPN > 200 (on 25-point scale) = blocking.** Must have an automated mitigation before the change merges.

### Worked example — Forge CLI

| ID | Threat | Component | STRIDE/ATLAS | Likelihood | Impact | RPN |
|---|---|---|---|---|---|---|
| T1 | Prompt injection via malicious `proposal.md` in untrusted branch | Forge plan pipeline | ATLAS: Initial Access | 3 | 4 | 12 |
| T2 | Secret leakage — Anthropic prompt includes raw secret value | Forge → Anthropic API call | I (STRIDE) | 2 | 5 | 10 |
| T3 | Orchestrator lock file corrupted by concurrent Forge runs | `~/.forge/locks/` | T (STRIDE) | 4 | 4 | 16 |
| T4 | CI runner gains write access beyond repo scope via PAT over-permission | CI → GitHub | E (STRIDE) | 2 | 5 | 10 |

---

## Q3 — What are we going to do about it?

> _One mitigation per identified threat. Choose: Lint rule / Verifier gate / Manual review / Accepted risk._

### Mitigation options

| Option | When to use |
|---|---|
| **Lint rule / static check** | Threat is detectable at PR time (path pattern, import, env var reference) |
| **Verifier gate** | Threat is detectable at build/test time (type-check, property-based test, schema validation) |
| **Manual review** | Threat requires human judgment; use `manual-review` label to enforce |
| **Accepted risk** | RPN < 5, or cost of mitigation exceeds impact; must be explicitly recorded |

### Mitigation table

| Threat ID | Mitigation | Type | Owner |
|---|---|---|---|
| T1 | _mitigation description_ | lint / verifier / manual / accepted | _owner_ |

### Worked example — Forge CLI

| Threat ID | Mitigation | Type | Owner |
|---|---|---|---|
| T1 | Forge plan pipeline sandboxes file reads; never auto-executes instructions from non-trusted markdown | Verifier gate | pavel |
| T2 | Forge prompt-builder validates no raw secret patterns (regex) before sending to Anthropic | Lint rule | pavel |
| T3 | Lock file operations wrapped in exclusive file lock; red-line enforced — orchestrator code is hand-coded only | Manual review (red-line) | pavel |
| T4 | PAT scoped to `repo` only; principle of least privilege documented in CI workflow | Manual review | pavel |

---

## Q4 — Did we do a good job?

> _Post-review checklist. Answered at PR review time._

- [ ] All T-rows with RPN > 10 have a mitigation assigned.
- [ ] No HIGH-RPN entries (RPN > 200 on 25-point scale) remain open.
- [ ] Peer review completed (Pavel + 1, or Pavel solo if sole contributor).
- [ ] Relevant items from [`security-review-checklist.md`](security-review-checklist.md) are ticked.
- [ ] Tests pass (unit + type-check + lint).
- [ ] Any "accepted risk" entries are documented with rationale and will be added to [`_root/risk-register.md`](../../_root/risk-register.md).

---

## LINDDUN GO requirement for capabilities with PII surface (applied 2026-05-13 from change `validate-personal-data-security`)

Capabilities that introduce or materially extend a **PII surface** (user-identifiable data fields, cross-tenant data flows, external processor integrations, or new data collection surfaces) MUST also complete a **LINDDUN GO pass** in addition to the Shostack 4Q frame.

**LINDDUN GO** is the lightweight 7-category privacy threat model (Linkability, Identifiability, Non-repudiation, Detectability, Disclosure of information, Unawareness, Non-compliance). The 7×N matrix format (category × data flow) is documented in the worked exemplar at `_methodology/security/canvas-permission-linddun.md`.

**When the requirement triggers:**
- New `auth.users` metadata fields or new `app_metadata` flags.
- New tables containing user-identifiable data (email, IP, user content).
- New external processors (Anthropic, Cloudflare sub-product, third-party integrations).
- New cross-boundary data flows (e.g., mirror export, DSAR extraction path, webhook outbound).

**When it does NOT trigger:** Internal refactors, UI copy changes, spec-only changes, changes with no new PII data flow.

Add a `## LINDDUN GO summary` section to `design.md` when the trigger applies. A full artefact (threat catalogue + matrix) is required only when the threat surface is novel — reference the exemplar for the format.

---

## References

- [Adam Shostack — 4-Question Frame](https://github.com/adamshostack/4QuestionFrame)
- [MITRE ATLAS](https://atlas.mitre.org/)
- [STRIDE overview — OWASP](https://owasp.org/www-community/Threat_Modeling_Process)
- [`security-review-checklist.md`](security-review-checklist.md) — OWASP LLM Top 10 PR checklist
- [`_root/risk-register.md`](../../_root/risk-register.md) — cross-epic risk log
- [`../risk-taxonomy.md`](../risk-taxonomy.md) — L2 Security level definition
