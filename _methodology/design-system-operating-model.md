# Design System — Operating Model

> Status: drafted in `define-design-system-spec-convention`. Promotes to `openspec/_methodology/design-system-operating-model.md` at archive (Phase 6.5).
>
> Companion to [`design-system-convention.md`](design-system-convention.md), which governs *what* the corpus looks like; this file governs *how* it changes.

---

## 1. Roles

Four roles. Each names one human or one programmatic actor and locks ownership boundaries. Boundaries are deliberate — collapsing two roles into one merges blast radii and produces specs that fail review for governance reasons rather than content reasons.

| Role | Owns | Does not own | Decision authority |
|---|---|---|---|
| **Design System Lead** (Pavel) | Governance; D-DS-N ratification; audit-finding triage; convention amendments; refresh-trigger calls | Day-to-day spec authoring; code implementation | Final on every governance question; sole signing authority for `ratified` status |
| **Design Agent** | drafts/spec-prose authoring; Findings with path:line citations; migration plans; governance-proposal drafting; surfacing Q-DS-N | Writes to `current/` outside archive; ratification of D-DS-N; code-side implementation | Autonomous: register Q-DS-N during work; raise stop-conditions; propose migration shape |
| **Code Agent** | Implementation against an approved spec; code-side migrations; Tailwind / `@theme` wiring; component code; tests | Spec mutation; governance | None over spec; raises spec-conflict findings to Design Agent |
| **Investigator Agent** (read-only Code) | Audit execution; drift detection; factual research; spec-vs-impl reconciliation reports | Any write to repo; any spec mutation; any decision | None; output is a report, consumed by Design Agent (or by Pavel directly when finding is in an archived change) |

The Investigator Agent's role-shape — separate Claude session in Code-mode with read-only tools, vs. sub-tool inside Code Agent — is open as **Q-DS-5**. Current leaning: separate session.

### 1.1 Investigator trigger conditions

Investigator runs only when triggered, not on a schedule of its own.

- Pavel requests factual research for a governance decision.
- Design Agent declares a claim outside the provided context (e.g. "shadcn does X") and asks for verification.
- Quarterly audit launches per [`design-system-audit-protocol.md`](design-system-audit-protocol.md).
- Doubt about a specific fact during change-authoring.
- **Investigator finds a discrepancy in an archived change.** In this case the output goes **directly to Pavel** (not via Design Agent); Pavel decides whether a correcting change is required.

---

## 2. Workflow — eight steps

A finding becomes an archived change through this sequence. Each step names the owner, the artefact produced, and the checkpoint that gates the next step.

| # | Step | Owner | Artefact | Checkpoint |
|---|---|---|---|---|
| 1 | **Detection** | anyone (incl. audit) | Finding note (severity + location) | Finding has path:line citation |
| 2 | **Triage** | Pavel | Triage decision | S0 → change opens immediately; S1 → next sprint; S2 → registered as Q-DS-N |
| 3 | **Research** | Investigator | Research note (facts with citations) | Every claim has path:line or URL |
| 4 | **Validation** | Pavel | Go/no-go on scope | Scope confirmed; budget agreed |
| 5 | **Authoring** | Design Agent | `proposal.md` + `design.md` + `tasks.md` + `agent-brief.md` | Four files exist; every Finding is cited; R1..R13 self-check passes |
| 6 | **Review** | Pavel | Approval or revision request | Approval is recorded explicitly; revisions iterate at step 5 |
| 7 | **Implementation** | Code Agent | PR(s) | Tests + lint + typecheck pass; PR title and body reference change ID |
| 8 | **Archive** | Design Agent | Archive folder + applied-delta footer in `current/` | `openspec/changes/<slug>/` renamed under `openspec/archive/YYYY-MM-DD-<slug>/`; capability spec carries `## <Title> (applied YYYY-MM-DD from change <slug>)` paragraph |

Steps 5 and 6 may iterate. Steps 7 and 8 do not — once authored, the artefact does not mutate; corrections go through a follow-up change.

---

## 3. Decision authority matrix

What can each role decide unilaterally? What requires a change-proposal? The matrix is the operational complement to **R6** ("cross-cutting decisions go in `decisions.md`, not scattered through prose").

| Operation | Pavel solo | Change required | Design Agent autonomous |
|---|---|---|---|
| OKLCH value tweak in `globals.css` | ✅ (per R1 — values are not specs) | | |
| Add new R-rule (R14+) to convention | | ✅ | |
| Surface new Q-DS-N during work | | | ✅ |
| Mark spec section deprecated | | ✅ | |
| Typo / broken-link fix in `current/` | | | ✅ (per `studio-charter.md § 5.3` exception) |
| Add new capability spec file | | ✅ | |
| Register D-DS-N | | ✅ (within parent change; ratified by Pavel at archive) | |
| Quarterly audit triage | ✅ | | |
| Add new role to operating model | | ✅ | |
| Run ad-hoc Investigator query | ✅ | | (or Design Agent via tool-call) |
| **Reject a ratified D-DS-N (supersede)** | | ✅ (new change) | |

The last row is critical: ratified decisions are not edited. They are superseded by a new change that registers a successor `D-DS-N` referencing the prior. The historical entry keeps `ratified` status but gains a `supersededBy` field on amendment.

---

## 4. Refresh triggers

The convention is not static. Five conditions trigger a convention review:

1. **Tailwind major version** (v5+). `@theme` semantics may change; resolution order, utility-naming, or directive shape may shift. **R4** may need rewording.
2. **W3C DTCG major version**. `$type` closed list may grow; composite-type semantics may change. **R11** may need expansion.
3. **New surface added** to the studio (mobile, native, embed, deck). Theme-level boundaries shift; **R13** may need a new theme name.
4. **WCAG major version update**. A11y baseline (**R9**) may need expansion (e.g., new contrast targets, new touch-target rules).
5. **Annual review** — twelve months from the last convention amendment. Even with no triggering event, the review checks for accumulated drift between R1..R13 and lived practice.

---

## 5. Anti-patterns

These are operating-model failures, distinct from the per-file content anti-patterns in [`design-system-convention.md § Anti-patterns per file`](design-system-convention.md). When seen in practice, they are findings of severity **S1** minimum.

1. **Single-source-of-failure agent.** A single Claude session holding all four agent roles simultaneously. Roles are bounded for blast-radius reasons; merging them collapses the boundary.
2. **Plan-as-instruction-list.** Treating the workflow's step list as a checklist to mechanically tick. The workflow is a contract for *what produces what*; the work itself remains thinking, not ticking.
3. **Context bloat without compaction.** Letting a change-authoring session accumulate hundreds of K of tool output without snipping settled work. Diminishes the agent's working memory and produces inconsistent specs.
4. **Governance-as-compliance.** Treating R1..R13 as paperwork to satisfy rather than as constraints that produce coherent specs. Compliance produces specs that pass Verification but fail review.
5. **AI-driven cleanup without human approval.** An audit finds a "fix"; an agent applies it directly to `current/` without a change-proposal. Violates `studio-charter.md § 9 Hard rule 7`.
6. **Chat-history coupling.** Briefs and specs that reference chat-message IDs (e.g. `m0049`, `m0051`) instead of fixed artefacts. Through six months in archive, message IDs are unreadable. Briefs and specs reference: **decisions** by ID (`D-DS-N`), **source files** by `path:line`, **industry standards** by URL, **open questions** by ID. If a decision was reached in chat but not registered in `decisions.md`, it is a governance gap — not a source.

---

## 6. Cross-references

- [`design-system-convention.md`](design-system-convention.md) — what the corpus looks like (R1..R13).
- [`design-system-audit-protocol.md`](design-system-audit-protocol.md) — quarterly audit format, triage, severity scale.
- [`templates/design-system/`](templates/design-system/) — literal skeletons for proposal / design / tasks / agent-brief.
- [`studio-charter.md`](studio-charter.md) § 5 (OpenSpec methodology) and § 9 (Hard rules) — upstream governance this model implements.
