# Inquiry — Spec-code traceability: feasibility and CI pipeline design

> **Status:** scoping
> **Lead:** human:pavel
> **Owner:** Pavel Rapoport <pavel@rapoport.studio> (founder, primary)
> **Authors:** Pavel Rapoport <pavel@rapoport.studio> (primary); Claude Opus 4.7 (Anthropic) — desk research, synthesis (2026-05-13 parallel agent run)
> **Opened:** 2026-05-13
> **Capability_refs:** forge, inquiry
> **Thesis:** A coverage × inverse-drift metric computed via SCIP-derived symbol→spec-line tuples achieves sub-500ms CI latency without LLM involvement on the hot path, making spec traceability a feasible CI gate at the Rapoport Studio scale.
> **Method:** Single-pass desk research — synthesis of SCIP tooling documentation, semantic code indexing literature, and CI performance benchmarks. Adjacent evidence from SWE-bench Pro scaffold-spread datapoint (arxiv 2509.16941). ~15 sources evaluated May 2026.
> **Reusable for:** Forge audit-function design; drift-coverage pipeline proposal; future `search_inquiry()` tool specification. Authorship: open.
> **Companion to:** [`inquiry-context-as-bottleneck.md`](./inquiry-context-as-bottleneck.md), [`inquiry-codegen-fidelity-metrics.md`](./inquiry-codegen-fidelity-metrics.md), [`spec-to-code-validity-2026.md`](./spec-to-code-validity-2026.md)
> **Feeds into:** `openspec/changes/add-inquiry-entity/` (drift-coverage dependency §5); Forge `search_inquiry()` tool (Phase 3)

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

The drift-coverage dependency documented in `add-inquiry-entity/design.md § 5` requires structured `(capability_ref, finding_id, evidence_snippet, confidence)` tuples to compute:

```
coverage = lines with ≥1 finding / total lines
drift    = lines with 0 findings / total lines
```

The key question this inquiry addresses: is the symbol→spec-line resolution step (SCIP + semantic anchors) performant enough to run as a CI gate, or does it require batch/async execution?

The 2026-05-13 synthesis examined:
1. **SCIP tooling maturity** — what languages are supported; what the index latency looks like at repo scale.
2. **Semantic anchor design** — how spec-line anchors survive edits to the spec markdown (anchor stability problem).
3. **LLM-free hot path** — whether the tuple extraction step can be purely structural (grep + SCIP) vs. requiring an LLM to resolve ambiguous mappings.
4. **Falsifier**: if the CI latency exceeds 500ms per PR at the current repo scale, the on-path approach is infeasible → batch/async redesign.

---

## Open questions

1. What is the minimum anchor convention that survives spec-line renames without breaking all downstream tuples?
2. At what corpus size does the SCIP index become the bottleneck vs. the tuple-join step?
3. Does the LLM-free hot path produce enough false-positives on ambiguous symbol→spec mappings to invalidate the metric?
