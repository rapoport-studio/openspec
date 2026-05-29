# Inquiry — Codegen fidelity metrics: spec-traceable quality measurement

> **Status:** scoping
> **Lead:** human:pavel
> **Owner:** Pavel Rapoport <pavel@rapoport.studio> (founder, primary)
> **Authors:** Pavel Rapoport <pavel@rapoport.studio> (primary); Claude Opus 4.7 (Anthropic) — desk research, synthesis (2026-05-13 parallel agent run)
> **Author_ids:** human:pavel, agent:anthropic:claude-opus-4-7
> **Opened:** 2026-05-13
> **Capability_refs:** forge, inquiry
> **Thesis:** Spec-driven generation fidelity is measurable as a compound score (spec-coverage × confidence-floor × non-revert rate) traceable to individual capability specs; current industry benchmarks measure task success but not spec lineage.
> **Method:** Single-pass desk research — synthesis of SWE-bench Pro, HumanEval, MBPP, and emerging spec-traceability literature. Examined how existing benchmarks define "correct" and where they fail to capture spec-alignment. ~12 sources evaluated May 2026.
> **Reusable for:** Forge audit-function design; drift-coverage pipeline specification; future capability-quality reporting. Authorship: open.
> **Companion to:** [`inquiry-spec-code-traceability.md`](./inquiry-spec-code-traceability.md), [`inquiry-context-as-bottleneck.md`](./inquiry-context-as-bottleneck.md), [`forge-epic-first-run-results.md`](./forge-epic-first-run-results.md)
> **Feeds into:** `openspec/archive/2026-05-29-add-inquiry-entity/` (drift-coverage pipeline §5); Forge `search_inquiry()` tool (Phase 3); capability audit tooling

---

## Triage

| # | Kind | Contributor | Status | Note |
|---|---|---|---|---|
| — | — | — | — | No triage entries yet — Phase 1 scoping only |

---

## Findings

_No promoted findings yet. Promotion requires explicit triage decision (Phase 2+)._

---

## Background

Current codegen benchmarks (SWE-bench, HumanEval, MBPP) measure whether generated code passes tests — they do not measure whether the code is traceable to a specification. For a spec-driven studio, test passage is necessary but not sufficient: a correct implementation that violates a spec line is still a spec violation.

The 2026-05-13 synthesis examined this gap and proposed a compound fidelity score:

```
fidelity(PR) = spec_coverage(PR) × confidence_floor(PR) × non_revert_rate(PR)

where:
  spec_coverage    = fraction of changed lines with ≥1 InquiryFinding.capability_ref match
  confidence_floor = min(confidence) across matched findings for this PR
  non_revert_rate  = 1 - (reverted_PRs / total_PRs) rolling 30-day window
```

This metric is defined per-PR and per-capability, making it auditable at the granularity of individual spec sections.

**Why three components:**
- `spec_coverage` alone rewards superficial tracing (any finding counts).
- `confidence_floor` penalizes PRs that pass only low-confidence findings.
- `non_revert_rate` is the ground-truth signal: if code built on a finding keeps getting reverted, the finding was wrong.

**Falsifier:** if the compound score does not correlate with subjective "spec-fidelity" ratings on a 20-PR sample, the metric definition is wrong — redesign before wiring into CI.

---

## Open questions

1. Is confidence_floor the right aggregation, or should it be confidence_mean (more stable) or confidence_max (more optimistic)?
2. What is the baseline non_revert_rate for the current Forge-generated PRs? Without a baseline, the metric has no reference point.
3. Does `spec_coverage` need to distinguish between "spec line referenced in a finding" vs. "spec line directly cited in the PR description"? The latter is easier to compute but less rigorous.
