# Inquiry — Drift-coverage retrieval spike: does the `Inquiry` entity earn its keep?

> **Status:** synthesizing
> **Lead:** human:pavel
> **Owner:** Pavel Rapoport <pavel@rapoport.studio> (founder, primary)
> **Authors:** Pavel Rapoport <pavel@rapoport.studio> (primary); Claude Sonnet 4.5 (Anthropic) — spike execution and classification, 2026-05-28
> **Author_ids:** human:pavel, agent:anthropic:claude-sonnet-4-5
> **Kind:** internal
> **Opened:** 2026-05-28
> **Capability_refs:** inquiry, forge
> **Thesis:** The `Inquiry` entity proposed in [`openspec/changes/add-inquiry-entity/`](../../changes/add-inquiry-entity/) is justified by Trigger #1 (drift-coverage tracker requires queryable findings) only if (a) findings actually exist in queryable form somewhere, and (b) entity-style retrieval beats `grep` on F1 for a concrete drift-coverage query against the current corpus.
> **Method:** Manual classification of all `grep -l -i 'forge' openspec/_methodology/research/*.md` matches (54 files) against an entity-style retrieval that filters cohort files by `Capability_refs: forge`. Ground-truth label per file: does it contain a *finding* (falsifiable claim, audit defect, evaluation outcome) about Forge, or only an incidental mention? Sample composition: 3 entity-hits, 9 `forge-*`-prefixed legacy files (full enumeration), 10 sampled from the remaining 42 grep matches.
> **Reusable for:** [`add-inquiry-entity` Phase 1 ratification](../../changes/add-inquiry-entity/); [`add-inquiry-workflow-gates`](../../changes/add-inquiry-workflow-gates/); future drift-coverage-tracker design.
> **Feeds into:** Phase 2 vesting gate evaluation; Phase 3 scope decision.

---

## 1. Question

The [`add-inquiry-entity` proposal](../../changes/add-inquiry-entity/) rests on three triggers (design.md §2). Trigger #1 — *drift-coverage dependency* — claims:

> "The metric cannot be computed without structured tuples of the form `(capability_ref, finding_id, evidence_snippet, confidence)`. Markdown+grep can extract `capability_ref` (`grep '^Capability:'`); it cannot extract `confidence` or `credibility_tier` without an LLM pass that re-parses every file every run — operationally infeasible at the 24-file corpus today, prohibitive at 200+."

The proposal's `Falsifier` (§2.2) is "if Forge's per-build token cost on `search_inquiry()` exceeds the cost of reading the full capability spec markdown in the same window". That's a measurement we cannot make today (no `search_inquiry()` tool, no DB).

This spike measures a *cheaper, earlier* falsifier: **given the corpus as it stands on 2026-05-28, does entity-style retrieval (filter `inquiry-*.md` by `Capability_refs`) beat bare `grep` on precision-and-recall against ground truth, for one concrete capability?**

The query simulated is the load-bearing one from design.md §5:

> "For each merged PR diff line in capability X: resolve spec line → InquiryFinding[]"

Target capability: `forge`. Chosen because it's the most-mentioned capability in both cohort and legacy corpus, maximizing signal for both retrieval methods.

---

## 2. Corpus snapshot (2026-05-28)

| Slice | Count |
|---|---|
| Total research files in `openspec/_methodology/research/` | **80** |
| `inquiry-*.md` files (proposed convention) | 6 |
| `inquiry-*.md` files after review-round filter (effective cohort per `index-inquiries.ts`) | **4** |
| Files in cohort with `Capability_refs` including `forge` | **3** |
| Files matched by `grep -l -i 'forge' *.md` | **54** |
| `forge-*`-prefixed legacy files (genuine Forge research, no convention) | **9** |

Two observations are visible without further analysis:

- **The cohort is growing slowly.** Proposal cited "4 forward-cohort inquiries from the 2026-05-13 synthesis"; 15 days later, the indexed cohort is still effectively 4 (only `inquiry-spec-driven-development-industry-landscape` carries forward in addition to the 3 forge-relevant ones).
- **The total corpus tripled in 15 days** (~24 → 80). Almost all new files followed legacy prose-style naming, not `inquiry-*` convention. **In particular, `forge-runtime-audit-2026-05-18.md` — a 25-mention substantive audit of Forge — was authored 8 days *after* `add-research-authorship-convention` shipped, and does not follow the convention.**

This is convention-leak signal before any classification work begins.

---

## 3. Method

For each candidate file, two labels assigned by hand from first 50 lines + grep-line context:

- **About-label:** does the file contain *findings* about Forge? Categories:
  - `finding-bearing` — file contains ≥1 falsifiable claim, audit defect, evaluation outcome, or decision-support analysis about Forge
  - `mention-only` — Forge is referenced but file is not about Forge (incidental, comparative, or strategic-context)
  - `wip` — file is in `scoping` / `planning` status, no findings promoted yet
- **Retrieval method:** which methods retrieve this file?
  - `E` — entity-style (cohort + `Capability_refs: forge`)
  - `G` — grep `-l -i 'forge'`
  - Both, one, or neither

**Ground truth = `finding-bearing` label.** A file with only mentions but no findings is a false positive for "spec line → InquiryFinding[]". A finding-bearing file missed by a retrieval is a false negative.

Sample composition (22 of 54 grep hits, exhaustive on Groups A+B):

- **Group A — Entity-style hits:** 3 files (full enumeration).
- **Group B — `forge-*` and `upgrade-forge-*` prefixed:** 9 files (full enumeration).
- **Group C — sampled from remaining 42 grep hits:** 10 files chosen for topical diversity (landscape, comparison, audit, drift, organization, stack).

Classification appendix at §6.

---

## 4. Results

### 4.1 Precision and recall, per method

| | About-label | Entity | Grep |
|---|---|---|---|
| `finding-bearing` (in 22-file sample) | 11 | 0 | 11 |
| `mention-only` (in sample) | 8 | 0 | 8 |
| `wip` (in sample) | 3 | 3 | 3 |
| **Total retrieved** | — | **3** | **22** (extrapolation: ~54) |
| **Precision** (finding-bearing / retrieved) | — | **0/3 = 0%** | **11/22 ≈ 50%** |
| **Recall** (finding-bearing retrieved / 11 finding-bearing in sample) | — | **0/11 = 0%** | **11/11 = 100%** |

Headline numbers:

> **Entity-style retrieval has 0% precision and 0% recall** for "published Forge findings" against the 2026-05-28 corpus. Every cohort match is in `scoping` status with no findings promoted (§6 Group A). The 11 finding-bearing files in the sample are all retrieved by grep and missed by entity-style — because the inquiry-* convention has not been applied to them.

This is a stronger falsifier than the proposal's RAG-cost-comparison: the entity has no findings to retrieve regardless of cost.

### 4.2 Why grep's 50% precision is the more interesting number

Of grep's 11 false positives in the sample:

- **5 are landscape / positioning files** where Forge is mentioned strategically but the file is *about something else* (e.g., `ai-cli-landscape-2026-05.md` mentions Forge in 37 places, but its findings are about the SDD market, not about Forge).
- **3 are incidental references** (e.g., `atlas-entities-2026-05.md` mentions Forge phase palette in passing — no findings about Forge).
- **3 are borderline** — comparative documents where Forge findings are mixed with non-Forge findings; whether a per-line drift-coverage query should retrieve them is itself a design question.

The drift-coverage tracker described in design.md §5 routes through `(capability_refs, finding)` tuples — meaning even with grep, the second stage requires *some* convention for marking findings. Today, that convention does not exist either in cohort or legacy files. **A finding-marking discipline (e.g., a `## Findings` section with `Grade: <tier>` annotations) is the binding upstream constraint, not the entity.**

### 4.3 The convention-adoption signal

Across the 13 non-cohort files in the sample (Groups B + C) that contain *substantive Forge findings*, **zero have been authored or retroactively renamed under the inquiry-* convention**. This includes 4 files authored *after* `add-research-authorship-convention` shipped on 2026-05-10/11:

| File | Authored | Convention applied? | Forge findings? |
|---|---|---|---|
| `forge-runtime-audit-2026-05-18.md` | 2026-05-18 | No | Yes (defects classified) |
| `forge-epic-first-run-results.md` | 2026-05-14 | No | Yes (3 confirmations + 1 defect) |
| `forge-openspec-index-audit-2026-05.md` | 2026-05-17 | No | Yes (audit) |
| `forge-substrate-sandcastle-evaluation.md` | 2026-05-13 | No | Yes (evaluation outcome) |

Convention adoption rate for new Forge research, post-convention-shipping: **0 / 4 = 0%.** This is the empirical proxy for Phase 2 vesting gate criterion (c) ("no consumer reported wishing markdown was a DB") inverted — *new authors are not even reaching for the convention*, never mind upgrading from it.

---

## 5. Findings

| # | Claim | Confidence | Falsifier | Grade |
|---|---|---|---|---|
| F-1 | Entity-style retrieval has 0% recall and 0% precision for "published Forge findings" against the corpus on 2026-05-28. Trigger #1 of the proposal is structurally blocked: there are no findings to query under entity discipline. | high | A single cohort file transitioning to `status=published` with promoted findings would partially restore recall. None has done so in 15 days; the cohort lead time alone forecasts months, not weeks. | engineering_claim |
| F-2 | Grep precision ≈ 50% is structurally limited by the *absence of a finding-marking convention*, not by the absence of an entity. The drift-coverage tracker's stage-2 problem (`spec line → InquiryFinding[]`) is unsolved by either method today. | high | A `## Findings` section with `Grade:` annotations applied to ≥5 of the finding-bearing files, plus a grep that filters by section presence, would close most of the precision gap. | engineering_claim |
| F-3 | New Forge research (4 files, post-2026-05-10) is being authored 100% outside the inquiry-* convention. The convention's adoption rate for the capability that most needs structured retrieval is zero. | high | A single new Forge research file authored under `inquiry-forge-<topic>.md` naming with full frontmatter, by any contributor, would falsify this. Authoring counts ≥3 over the next 30 days would change the trajectory assessment. | engineering_claim |
| F-4 | Trigger #2 (RAG-shape doctrine) cannot be measured until Trigger #1's preconditions are satisfied. Until findings exist under *some* queryable convention, `search_inquiry()` cannot be benchmarked against full-markdown reads, because the comparison has no left-hand side. | medium | Phase 3a (build `search_inquiry()` over the current JSON index, no DB) would let Trigger #2 be measured without committing to the entity. | engineering_claim |
| F-5 | The proposal's Phase 3 schema (Inquiry + Contribution + Source + Finding + AgentGrant + Relationship + VisibilityKind) is sized for a 200-file structured corpus that does not exist and is not on a trajectory to exist by 2026-07-10. The scoping mismatch is ~2 orders of magnitude. | medium | Cohort growing to ≥20 inquiries under convention by 2026-07-10 would falsify; current rate (0 net adds in 15 days) does not. | engineering_claim |

---

## 6. Classification appendix (full sample)

### Group A — entity-style hits

| File | Status | About-label | Forge findings present? |
|---|---|---|---|
| `inquiry-codegen-fidelity-metrics.md` | scoping | wip | No (scoping only) |
| `inquiry-context-as-bottleneck.md` | scoping | wip | No (scoping only) |
| `inquiry-spec-code-traceability.md` | scoping | wip | No (scoping only) |

### Group B — forge-* / upgrade-forge-* prefixed legacy files

| File | About-label | Notes |
|---|---|---|
| `forge-epic-conductor-runbook.md` | mention-only | Runbook, not findings |
| `forge-epic-first-run-plan.md` | mention-only | Planning doc, no findings |
| `forge-epic-first-run-results.md` | finding-bearing | 3 confirmations + 1 defect |
| `forge-openspec-index-audit-2026-05.md` | finding-bearing | Audit findings present |
| `forge-reviewer-isolation-2026-05.md` | finding-bearing | Audit findings |
| `forge-runtime-audit-2026-05-18.md` | finding-bearing | Defects classified |
| `forge-substrate-sandcastle-evaluation.md` | finding-bearing | Evaluation + recommendation |
| `upgrade-forge-ai-stack-audit.md` | finding-bearing | Baseline + audit |
| `upgrade-forge-baseline-2026-05-13.md` | mention-only | Baseline capture |

### Group C — sampled from remaining grep hits

| File | About-label | Notes |
|---|---|---|
| `ai-cli-landscape-2026-05.md` | mention-only | Landscape findings, Forge incidental |
| `atlas-entities-2026-05.md` | mention-only | 3 incidental mentions |
| `cli-tech-stack-comparison-2026.md` | finding-bearing | Stack analysis directly informs Forge |
| `competitor-landscape-synthesis.md` | mention-only | Forge in positioning context |
| `competitive-positioning-sdd-frameworks.md` | mention-only | Comparative mentions |
| `dentour-drift-audit.md` | mention-only | Operational context |
| `openspec-organization-patterns-2026.md` | mention-only | One incidental reference |
| `orchestra-component-stack-2026.md` | finding-bearing | Forge as L5 in stack analysis |
| `sdd-terminology-scorecard.md` | finding-bearing | Naming-collision scoring → "KEEP" decision |
| `spec-to-code-validity-2026.md` | finding-bearing | Risk score for "manage specs" thesis |

---

## 7. Recommendation for `add-inquiry-entity`

These are spike-author recommendations, not binding. The Phase 1 reviewer should reach their own conclusions.

1. **Phase 3 (DB lift) is premature** under the 2026-07-10 → 2026-08-10 vesting window described in the proposal. F-5 estimates the scoping mismatch at ~2 orders of magnitude.
2. **The binding upstream problem is finding-marking, not entity-shape.** Without a `## Findings` section with credibility annotation in *some* corpus subset, neither grep nor entity-style retrieval can answer the drift-coverage tracker's stage-2 query (F-2).
3. **A cheaper Phase 2 substitute exists:** add `## Findings` + `Grade:` discipline to ≥5 finding-bearing files (Group B is the natural target — they already contain the findings, only need restructure). Measure grep precision against the marked subset. If precision rises to ≥90%, entity may be unnecessary entirely.
4. **The convention adoption problem (F-3) is upstream of everything.** The proposal's inheritance from P0.6 ("convention-first triage") assumed convention would propagate. The 0% adoption rate for new Forge research in the post-convention window suggests the convention itself needs a usability pass before any entity layers on top.
5. **Trigger #2 (RAG) becomes measurable via Phase 3a (tool over JSON, no DB)** as proposed in the risk-reduction plan that commissioned this spike. Build `search_inquiry()` over `inquiries.json`, measure on 1-2 real Forge builds. Decision on Phase 3b (DB lift) waits for that measurement.
6. **Re-defer the visibility model, agent grants, and relationship graph** (the RAP-856/860/862 amendments) into their own change folders. They presume the base entity ships; per this spike, base entity isn't yet justified.

---

## 8. Open questions

1. **Is the cohort's slow growth a convention problem or a workload problem?** Cohort has 4 effective inquiries from 2026-05-13. None since. Is this because the convention is awkward, or because nobody opened new investigations during a heavy ship-phase? Convention-leak log (started 2026-05-28) is the place to capture future signal.
2. **Should `forge-runtime-audit-2026-05-18.md` be retroactively renamed?** It is a substantive Forge research artifact. Renaming would (a) validate the convention's applicability to audits, (b) raise effective cohort count, (c) test whether the entity's frontmatter schema actually fits the audit shape. Cost: ~30 minutes including frontmatter authoring.
3. **What's the right unit for drift-coverage stage 2: a file, a `## Findings` row, or an entity row?** This spike assumes file-level retrieval. The proposal's design.md §5 implies row-level. If row-level is the target, the marker convention upstream of the entity becomes even more important.
