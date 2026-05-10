# Design System — Audit Protocol

> Status: drafted in `define-design-system-spec-convention`. Promotes to `openspec/_methodology/design-system-audit-protocol.md` at archive (Phase 6.5).
>
> Quarterly audits keep the design-system corpus honest between change-driven work. An audit produces one artefact and a triage decision; it does not, on its own, mutate `current/` or any spec.

---

## 1. Cadence

Quarterly. Auto-triggered by the Design Agent.

The trigger calendar is open as **Q-DS-6**:

- (a) **Calendar-fixed** — Q1=Mar, Q2=Jun, Q3=Sep, Q4=Dec.
- (b) **Rolling 90 days** from prior audit completion.
- (c) **Event-triggered** — 90 days since the last design-system spec merge.

Current leaning: **(c)**. Calendar audits run in vain when the system has not changed; rolling audits drift; event-triggered audits run only when there is something to audit.

Each audit produces exactly one artefact:

```
openspec/_methodology/audits/YYYY-Qn-design-system-audit.md
```

`YYYY` is the calendar year; `Qn` is `Q1`/`Q2`/`Q3`/`Q4`. The folder `_methodology/audits/` is created on the first audit; subsequent audits append files.

---

## 2. Audit scope

Audits cover **only**:

- Archived changes (`openspec/archive/`).
- Current state of `openspec/current/`.
- Methodology files in `openspec/_methodology/`.

**Excluded:** in-flight changes (`openspec/changes/<slug>/`). They are already under review through their own change-proposal flow. If an audit finding intersects with an in-flight change, the finding is recorded in **that change** as a cross-reference — it is not duplicated as a separate change.

This rule prevents audits from creating phantom work that competes with running change-streams.

---

## 3. Audit dimensions

Five concerns to check, in this order:

1. **Token drift.** Does every token declared in `design-tokens.md` exist in `globals.css`? Does every token in `globals.css` have a row in `design-tokens.md`? Are values, mode-pairs, and Tailwind utility mappings consistent?
2. **Naming inconsistency.** Are token names, component names, and role names used identically across `design-system.md`, `design-tokens.md`, `design-components.md`, `ui.md`? Are any aliases drifting from the reconciled mapping?
3. **Coverage gaps.** Does every component listed in `design-components.md` have a `lives_in` value? A `status`? Variant rationale? Token map? Are any spec sections missing required fields per R1..R13?
4. **Unused tokens.** Tokens declared in `design-tokens.md` that have zero observed consumers in `packages/ui` / `apps/studio` / `apps/web`. Candidate removals (Pavel decides).
5. **Accessibility regressions.** Does the contrast pair for every (foreground, background) token combination still meet WCAG AA at body size and AAA at large text? Has any focus-ring contract been violated?

---

## 4. Severity scale

Three-tier model adapted for design-system audits.

| Severity | Definition | Action |
|---|---|---|
| **S0** | Broken contract; blocks merge of dependent work | Open change immediately; cite finding in proposal |
| **S1** | Drift within a surface; not blocking; scheduled for next sprint | Open change in next sprint cycle; finding cited in `## 2. Findings` |
| **S2** | Informational; not blocking; possibly one-time noise | Register as Q-DS-N in `open-questions.md`; close audit without action |

S0 / S1 / S2 boundaries are deliberate: S0 is binary (either it blocks, or it does not), S1 is a budgeted commitment (next sprint, not "soon"), S2 is registered, not actioned.

This is the studio's own definition. There is no external citation.

---

## 5. Findings format

Each finding in an audit artefact is a row:

```
| # | Severity | Location              | Description                                | Proposed action          | Blocking? |
|---|----------|-----------------------|--------------------------------------------|--------------------------|-----------|
| 1 | S0       | design-tokens.md:128  | --color-canvas mapped to two utilities     | Open `fix-canvas-…`      | yes       |
| 2 | S1       | globals.css:412       | Orphan `--legacy-blueprint` no consumer    | Add to next sprint queue | no        |
| 3 | S2       | design-components.md  | SkillChip status open >90 days             | Register as Q-DS-N       | no        |
```

Required fields:

- `severity` ∈ {S0, S1, S2}
- `location` — file + line, or file + section name when line-specific reference does not apply
- `description` — one sentence, factual, no judgement
- `proposed action` — a concrete next step (open change `<slug>`, register Q-DS-N, add to sprint queue)
- `blocking?` — yes / no, derived from severity (S0 = yes, S1/S2 = no)

---

## 6. Triage

After the audit artefact is written, Pavel triages every finding:

- **S0** → opens (or assigns) a change folder; finding is the seed of the proposal's Findings section.
- **S1** → adds to the next sprint's queue. Change folder may not open immediately.
- **S2** → registers as Q-DS-N in `open-questions.md` and closes the audit.

Audit artefacts are append-only. Triage decisions are recorded in the artefact itself, on the same row, in a final column.

---

## 7. Output structure

Each audit artefact has these sections:

```
# YYYY Qn — Design System Audit

> Status: <draft / triaged / closed>
> Audited by: <Investigator Agent | Design Agent>
> Triaged by: Pavel
> Date range: YYYY-MM-DD — YYYY-MM-DD

## 1. Scope
## 2. Findings
## 3. Triage
## 4. Notes for the next audit
```

§ 4 is a free-form prose note: things to look at next quarter, instrumentation gaps, conventions that may need refreshing.

---

## 8. Cross-references

- [`design-system-convention.md`](design-system-convention.md) — R1..R13; audit checks each rule.
- [`design-system-operating-model.md`](design-system-operating-model.md) — Investigator Agent role and trigger conditions.
- [`studio-charter.md`](studio-charter.md) § 8 (capability status) — informs S0/S1/S2 mapping at the capability level.
