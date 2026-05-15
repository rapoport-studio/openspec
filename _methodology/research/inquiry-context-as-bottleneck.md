# Inquiry — Context as bottleneck: evidence and remediation

> **Status:** scoping
> **Lead:** human:pavel
> **Owner:** Pavel Rapoport <pavel@rapoport.studio> (founder, primary)
> **Authors:** Pavel Rapoport <pavel@rapoport.studio> (primary); Claude Opus 4.7 (Anthropic) — desk research, synthesis (2026-05-13 parallel agent run)
> **Author_ids:** human:pavel, agent:anthropic:claude-opus-4-7
> **Opened:** 2026-05-13
> **Capability_refs:** forge, cli-core, inquiry
> **Thesis:** LLM performance on multi-file engineering tasks is bounded by context engineering quality, not model scale; the 36-point scaffold advantage on SWE-bench Pro versus less than 1pt model-spread is the empirical load-bearing claim.
> **Method:** Single-pass desk research — synthesis of Chroma Research "context rot" benchmark (18 frontier models), Anthropic "Effective context engineering for AI agents" (Sep 2025), SWE-bench Pro scaffold-spread datapoint (arxiv 2509.16941), Answer.AI Devin reproduction (36 min vs 6h20m). ~20 sources evaluated May 2026.
> **Reusable for:** Forge context-engineering design; cli-core LLM-wrapping decisions; Muse prompt architecture; any agent scaffolding proposal. Authorship: open.
> **Companion to:** [`inquiry-spec-code-traceability.md`](./inquiry-spec-code-traceability.md), [`inquiry-codegen-fidelity-metrics.md`](./inquiry-codegen-fidelity-metrics.md), [`upgrade-forge-ai-stack-audit.md`](./upgrade-forge-ai-stack-audit.md), [`upgrade-forge-baseline-2026-05-13.md`](./upgrade-forge-baseline-2026-05-13.md)
> **Feeds into:** `openspec/changes/add-inquiry-entity/` (RAG-shape rationale §2.2); Forge `search_inquiry()` tool design; cli-core context-engineering primitives

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

The 2026-05-13 synthesis examined the "context-as-bottleneck" hypothesis across three evidence streams:

**Stream 1 — Empirical degradation evidence**

Chroma Research's 2025 "context rot" benchmark (18 frontier models, including Claude Opus 4 and GPT-4.1) showed every frontier model degrades F1 before its context-window limit. A 200K-token model can exhibit rot at 50K. Mechanisms: lost-in-the-middle, attention dilution, distractor interference.

**Stream 2 — Scaffold-spread vs. model-spread**

SWE-bench Pro results (arxiv 2509.16941) show a 36-point performance gap between best and worst scaffolds on the same model, versus <1pt gap across models on the same scaffold. This is the empirical falsifier for "just use a bigger model."

**Stream 3 — Remediation evidence**

Anthropic's "Effective context engineering for AI agents" (Sep 2025) and Answer.AI's Devin reproduction (36-minute open-source replication of a 6h20m claimed task) both point to the same remediation: **retrieval, not dump** — structured tool calls for relevant context, not full-context injection.

**Implication for `Inquiry`**

The RAG-shape doctrine (design.md §2.2) follows directly: `search_inquiry(capability_ref=X, credibility>=engineering_claim)` is a retrieval call that returns bounded findings rather than the full 90k-line bundle.

---

## Open questions

1. Is the 36-point scaffold-spread statistic durable — does it replicate on future SWE-bench versions, or is it a 2024-era artifact?
2. At what context length does the Forge Builder role exhibit measurable rot on multi-file tasks? (Falsifier: measure token cost per correctly-applied spec constraint.)
3. Does "retrieval, not dump" hold when the retrieval corpus itself is noisy (low-confidence findings mixed with high-confidence)?
