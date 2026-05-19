# 2026 Q2 — Design System Audit

> Status: draft
> Audited by: Design Agent
> Triaged by: Pavel
> Date range: 2026-05-19 — 2026-05-19

## 1. Scope

Event-triggered audit per `design-system-audit-protocol.md` § 1 trigger (c) — first audit since the design-system corpus came online. Surfaces a single S2 finding observed out-of-repo (Anthropic preview workspace) that has implications for the in-repo corpus.

In scope: methodology files in `openspec/_methodology/`, current corpus state in `openspec/current/`, the out-of-repo Anthropic preview workspace as a referenced artefact (read by Design Agent, not by Code Agent).

Out of scope: in-flight changes under `openspec/changes/<slug>/` (per audit-protocol § 2).

## 2. Findings

| # | Severity | Location | Description | Proposed action | Blocking? | Triage |
|---|---|---|---|---|---|---|
| 1 | S2 | n/a — out-of-repo Anthropic preview workspace | Workspace contains a design-system deliverable that mirrors `packages/ui/src/styles/{globals,patterns}.css` plus a marketing-kit recreation, vendored fonts, and brand-pattern adaptations. Mirror policy is unspecified — sync direction, refresh trigger, and source-of-truth disambiguation all open. | Register as Q-DS-8. | no | pending |

## 3. Triage

Pending Pavel triage. Per audit-protocol § 4: S2 → register as Q-DS-N and close the audit without further action. Awaiting Pavel's decision on Q-DS-8 to know whether a follow-up change folder (`sync-design-workspace-to-current` or `scope-icon-specialist-engagement`) opens.

## 4. Notes for the next audit

- First audit ever; observe whether event-triggered cadence (trigger (c)) holds, or whether Pavel re-opens Q-DS-6 to pick a different cadence.
- The corpus has no prior audit baseline. Subsequent audits can diff against this one.
- The five audit dimensions (audit-protocol § 3) were not exercised here — this audit was finding-driven, not dimension-driven. Next audit should sweep all five.
- Spec-vs-reality drift on D-DS-4: `openspec/_methodology/templates/` contains `_universal.yaml`, `closure-email-template.md`, `convention-adapter-example.md`, `legal-platform.yaml` — no `design-system/` subfolder, none of the four ratified templates (proposal / design / tasks / agent-brief) exist on disk despite D-DS-4 ratification and § 10 + operating-model § 6 cross-references. Treat as a candidate S1 finding next audit; do NOT action in this bootstrap.
- Canonical change-shape definition diverges across three governance sources: `openspec/AGENTS.md` (4 files with verification.md), `_methodology/studio-charter.md § 5.2` ("exactly three files"), `_methodology/design-system-operating-model.md § 2 step 5` (proposal+design+tasks+agent-brief). Latest archive `2026-05-18-clarify-muse-modes-framing` ships 3 files. AGENTS.md notes a "legacy-mode" carve-out but three documents are still mutually inconsistent on the canonical shape. Candidate S1 finding for the next audit's "Naming inconsistency" + "Coverage gaps" dimensions.
