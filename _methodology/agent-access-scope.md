---
title: Agent Access Scope
audience: [architect, executor]
---

# Agent Access Scope

Every agent in the Rapoport Studio orchestra declares its read/write surface as an **explicit allowlist**. Anything not declared is denied. This convention follows the capability-based (CapBAC) access model — the allowlist *is* the capability — rather than role-based inheritance. A reader auditing an agent spec should be able to determine its complete access surface from a single `## Access scope` section without tracing role hierarchies.

New tools and resources are **denied by default**. Adding a tool or path to an agent requires an explicit amendment to the agent's capability spec, not just to the implementation code.

This document defines the shape every `## Access scope` section must follow, lists the five mandatory clauses, and provides filled-in instances for the three Bootstrap-CLI-Orchestra agents (Forge, Muse, Maestro).

Prior art: [`openspec/current/agents/maestro.md § Access scope`](../current/agents/maestro.md). Research backing: [`openspec/_methodology/research/access-and-context-hierarchy-2026.md § Part 1`](research/access-and-context-hierarchy-2026.md).

---

## Template

An `## Access scope` section uses the following five-column allowlist table as its organizing unit. Repeat the table once per resource category (DB tables, file paths, external APIs). Tool permissions use a list form described below. Omit a category only when the agent has zero access to it — not when access is "not yet decided."

| Resource | Read | Write | Notes | Source of truth |
|---|---|---|---|---|
| _table name / path glob / service name_ | yes / no / _scope_ | yes / no / _constraint_ | _additional scope or exclusions_ | _canonical spec section or migration_ |

### Tool permissions (list form)

Tool names are not rows in the table above; they use an allowlist + denylist pair:

**Allowlist** — agent is permitted to invoke:
- _tool name_

**Denylist** — agent is not permitted to invoke. Future tool additions are **denied by default** unless explicitly added to the allowlist:
- _tool name_

### Why this list, not more

_One paragraph stating which broader capabilities were considered and deliberately excluded, and why the declared scope is sufficient. "Not needed for MVP" is a valid reason but must be written out explicitly. This paragraph converts an opaque list into an auditable decision._

---

## Mandatory clauses

Every `## Access scope` section in an agent capability spec must contain all five of the following:

**(a) Explicit DB-table allowlist.** List every database table the agent may read or write, with the exact operation (read / write / append-only write). Implicit access — "reads whatever tables the code touches" — is not permitted.

**(b) Explicit path allowlist with read/write distinction.** List every filesystem path or glob the agent may access. Read and write are separate columns; `write` includes create, edit, and delete unless otherwise stated. A path appearing in the Read column does not implicitly grant write.

**(c) Explicit external-API allowlist.** List every external service the agent may call (Anthropic, Linear, GitHub, Infisical, Supabase, Cloudflare, and any other). Include the access mode (read / write / none). Omission of a service means it is denied.

**(d) Explicit tool-name allowlist with deny-by-default rider.** List every tool the agent may invoke. The denylist must include the sentence: *"Future tool additions are denied by default unless explicitly added to the allowlist."* This sentence is what makes the allowlist a contract rather than a snapshot.

**(e) 'Why this list, not more' justification paragraph.** State which broader capabilities were considered and deliberately excluded. This paragraph is the human-readable complement to the denylist — it answers *why* the denylist is correct, not just what is on it, and is required for the section to pass spec review.

---

## Filled-in instances

The three tables below are copied verbatim from [`openspec/archive/2026-05-12-bootstrap-cli-orchestra-mvp/design.md § Access scope per CLI`](../archive/2026-05-12-bootstrap-cli-orchestra-mvp/design.md). They use a three-column Resource / Read / Write format — an MVP subset of the five-column template above; Notes and Source-of-truth columns will be filled in when each agent's full capability spec lands.

### Forge

| Resource | Read | Write |
|---|---|---|
| `openspec/current/` | yes (all) | no |
| `openspec/changes/<slug>/` | yes (assigned) | yes (code-side artifacts only — no proposal/design/tasks rewrite) |
| `openspec/archive/` | yes | no |
| `~/.rapoport/events/` | own only | yes (append) |
| Code (`apps/`, `packages/`) | yes | yes (via PR) |
| GitHub | PR/comment/issue read | PR open, issue comment |
| Linear | issue read | status, comment |
| Anthropic API | full | full |
| Infisical | `/forge/*` only | no |

### Muse

| Resource | Read | Write |
|---|---|---|
| `openspec/current/` | yes (all) | no |
| `openspec/changes/<slug>/` | yes (drafting) | yes (proposal.md, design.md, tasks.md only) |
| `openspec/archive/` | yes | no |
| `~/.rapoport/events/` | own only | yes (append) |
| Code | read for context | no |
| GitHub | issue/PR read | no (Pavel opens PRs) |
| Linear | issue read | no |
| Anthropic API | full | full |
| Infisical | `/muse/*` only | no |

### Maestro

| Resource | Read | Write |
|---|---|---|
| `openspec/current/` | yes (all) | no |
| `openspec/changes/` | yes (all) | no |
| `openspec/archive/` | yes | no |
| `~/.rapoport/events/` | all | no (read-only command set) |
| `~/.rapoport/lineage/` | yes | no (computed each call from events; no separate writes in MVP) |
| Code | no | no |
| GitHub | read PR/issue for context | no |
| Linear | read | no |
| Anthropic API | no (Maestro is deterministic in MVP — no LLM calls) | no |
| Infisical | `/maestro/*` only | no |

**Key MVP decision: Maestro has no LLM calls in v0.** It is a deterministic CLI that reads events.jsonl and formats output. Adding LLM-driven analysis is a v2 decision after data exists to analyze.

---

## Forward-reference convention

Per-CLI capability specs cite this document at their `## Access scope` heading rather than re-inlining these summary tables. The tables in this document are overview-level; the per-CLI spec sections are the authoritative, operationally-precise versions that add the full five-column form, tool-permissions list, and "why" paragraph.

Suggested forward-reference:

> _Access scope declared as an explicit allowlist following [`openspec/_methodology/agent-access-scope.md`](../../_methodology/agent-access-scope.md). Anything not declared is denied. Future tool additions are denied by default unless explicitly added to the allowlist below._
