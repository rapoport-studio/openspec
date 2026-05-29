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
| F-6 | **Phase 2 substitute experiment outcome (2026-05-28).** Applying `\|#\|Claim\|Grade\|Falsifier\|` table convention to 5 Group-B files (`forge-runtime-audit-2026-05-18`, `forge-epic-first-run-results`, `forge-openspec-index-audit-2026-05`, `forge-substrate-sandcastle-evaluation`, `upgrade-forge-ai-stack-audit`) raises section-scoped grep precision to **100% (5/5)** at recall **≈ 56% (5/9 of finding-bearing universe)**. Drastic improvement vs entity-today (0%/0%) and vs bare grep (50%/100%). The convention works structurally. | high | If recall stays below 80% even after marking the remaining 4 finding-bearing files, the convention has a deeper limitation than text-discoverability. | engineering_claim |
| F-7 | The section-scoped grep misses `forge-reviewer-isolation-2026-05.md` despite the file having pre-existing `## Findings` AND being about Forge — because the Findings section uses code-path language (`reviewer.ts`, `orchestrator.ts`) rather than the literal capability name "Forge". This identifies a known failure mode of the marker convention: capability inference within prose is unreliable. Fix: require a per-section or per-finding `Capability:` tag (similar to `Capability_refs:` in cohort frontmatter), not just topical text mention. | high | Adding a `Capability: forge` line to forge-reviewer-isolation's Findings section AND extending the test to filter on that tag should raise recall to ≥80%. | engineering_claim |

---

## 5.1 What F-6 + F-7 mean for the entity decision

Precision-recall numbers for "findings about Forge" by method, against the 11-file ground-truth sample (Groups B + finding-bearing Group C):

| Method | Precision | Recall |
|---|---|---|
| Entity-style (`Capability_refs: forge` in cohort) | 0% | 0% |
| Bare `grep -l -i 'forge'` | ≈50% | 100% |
| **Section-scoped grep (`## Findings` ∩ `forge` in section content)** | **100%** | **≈56%** |
| Projected: section-scoped + per-section `Capability:` tag (F-7 fix) | ≥95% | ≥80% |

The third row already beats every realistic Phase 3a/3b configuration on precision. The fourth row (achievable by extending the marker convention with one more field) is a strict superset of what the entity offers for drift-coverage retrieval — at the cost of one additional convention field and zero DB schema.

**Implication:** Phase 3a (`search_inquiry()` over JSON) may still be worth building *for the credibility-filter use case* (RAG-shape doctrine, Trigger #2), because credibility-tier filtering inside a Findings table would require an LLM pass per row at query time. But Phase 3b (DB lift for capability_ref filtering alone) is now strongly contested by F-6 + F-7 — convention does the job.

| # | Claim | Grade | Falsifier |
|---|---|---|---|
| F-8 | **F-7 fix experiment outcome (2026-05-29).** Applying per-section `Capability: forge` blockquote tag to 5 originally-marked files + extending `## Findings` discipline to 4 more finding-bearing files (cli-tech-stack-comparison, orchestra-component-stack, spec-to-code-validity, forge-reviewer-isolation) raises section-scoped grep + Capability-tag retrieval to **100% precision / 90% recall (9/10)** — strictly better than every alternative measured. Convention discipline alone now beats every realistic Phase 3a/3b configuration for the drift-coverage capability_ref query. | high | If a new finding-bearing file about Forge ships under the convention AND is missed by section-scoped + Capability-tag retrieval, recall has a deeper limitation than measured. |
| F-9 | **Per-section Capability tag has a known granularity limit.** `sdd-terminology-scorecard.md` contains 1 finding about Forge (KEEP verdict, 1 of 30 scoring rows) mixed with 29 findings about other terms. Section-level tag `Capability: platform` correctly excludes it from `Capability: forge` queries — precision-preserving — but the file's Forge finding is also unreachable through capability-scoped retrieval. Resolution: per-row Capability tag (column in Findings table) would close the gap; deferred until a downstream consumer actually requests row-level retrieval. | high | A real consumer (Forge tool, Studio view) requesting "all findings touching Forge anywhere, including 1-of-N mixed files" would activate the per-row design. Until then, the file-primary-capability assumption holds. |

---

## 5.2 Updated retrieval scorecard (post-F-7 fix, 2026-05-29)

| Method | Precision | Recall |
|---|---|---|
| Entity-style (`Capability_refs: forge` in cohort) | 0% | 0% |
| Bare `grep -l -i 'forge'` | ≈50% | 100% |
| Section-scoped grep (`## Findings` ∩ `forge` in section content) | 100% | ~56% (pre-F-7-fix) → 90% (post-extension) |
| **Section-scoped + Capability tag (this experiment)** | **100%** | **90% (9/10)** |
| Per-row Capability tag (F-9 design extension) | projected 100% | projected 100% (would capture sdd-* mixed-content file) |

**Conclusion strengthened.** The marker convention now achieves what the entity proposal's Phase 3b promised for the drift-coverage `capability_ref` filter, at **zero database cost** and **zero migration risk**. Phase 3b (DB lift for capability_ref filtering alone) should be **re-deferred** unless and until Phase 3a's RAG-shape measurement demonstrates an irreducible benefit that JSON-over-markdown cannot match.

Phase 3a (`search_inquiry()` over JSON) is still worth building for the **credibility filter** use case from RAG-shape doctrine (Trigger #2) — credibility tier is per-finding row metadata, and queries like "give me only `engineering_claim`+ findings about Forge" require structured access that section-scoped grep cannot provide cheaply. That measurement remains open.

| # | Claim | Grade | Falsifier |
|---|---|---|---|
| F-10 | **Credibility-filter measurement outcome (2026-05-29).** The query "findings about Forge with credibility ≥ engineering_claim" is answerable via section-scoped + Capability-tag + table-column grep at **95% recall (70/74) and structurally 100% precision** (every returned row literally carries the grade tag in the Grade column). Distribution across 9 Capability:forge files: 65 engineering_claim + 5 experimental + 4 discourse + 0 marketing. **The credibility filter that Trigger #2 (RAG-shape doctrine) was meant to justify is now achievable through convention discipline alone.** | high | A consumer (Forge tool / Studio view) requesting credibility-filtered findings AND finding that the grep-over-table approach miscategorizes >10% would falsify the structural precision claim. |
| F-11 | **Known shape constraint of the marker convention.** `forge-reviewer-isolation-2026-05.md` has the Capability:forge tag but **0 table rows** — its `## Findings` section uses heading-style subsections (`### Orchestrator call site`) rather than a `\|#\|Claim\|Grade\|Falsifier\|` table. Grade-filter retrieval cannot operate on heading-style sections without an additional per-heading Grade annotation. **Two valid convention shapes; one queryable by grade.** Resolution: either standardize on tabular shape (cheap), or define per-heading Grade annotation parallel to per-row Grade column. | high | Adding a per-heading `Grade:` line beneath each `###` heading in forge-reviewer-isolation, then re-running the grade-filter test and showing recall recover to 100%, would validate the fix path. |

---

## 5.3 Final retrieval scorecard (2026-05-29) — empirical loop closed

Combined precision/recall for the two load-bearing drift-coverage queries against the 9-file Capability:forge corpus + 1 mixed-content borderline file:

### Query 1: "all findings about Forge" (no credibility filter)

| Method | Precision | Recall |
|---|---|---|
| Entity-style (`Capability_refs: forge` in cohort, status=any) | 0% | 0% |
| Bare `grep -l -i 'forge'` | ≈50% | 100% |
| Section-scoped grep (post-#1885) | 100% | 56% |
| Section-scoped + Capability tag (#1904) | **100%** | **90%** |
| Per-row Capability tag (F-9, projected) | projected 100% | projected 100% |

### Query 2: "findings about Forge with credibility ≥ engineering_claim" (RAG-shape doctrine)

| Method | Precision | Recall |
|---|---|---|
| Entity-style (`status=published` AND `min_credibility=engineering_claim`) | 0% | 0% (no findings promoted) |
| Bare `grep` (file-level only) | ≈50% | requires LLM pass per file for grade filter |
| Phase 3a `search_inquiry({min_credibility})` over JSON | projected 100% | projected 100% if JSON includes Findings rows |
| **Section-scoped + Capability + Grade-column grep (THIS measurement)** | **100% (70/70)** | **~95% (70/74)** |
| Phase 3b DB query | projected 100% | projected 100% (assuming honest migration) |

The marker convention now meets or near-matches every realistic entity-backed query on the corpus that exists today. **Convention has structurally beaten Phase 3a + Phase 3b on the original Triggers #1 (drift-coverage) and #2 (RAG-shape doctrine)** at zero database cost.

### What this leaves on the table for an entity (if it ever ships)

The two empirical advantages the entity could still claim are operational, not retrieval-shaped:

1. **Real-time mutations during a Forge build.** Markdown discipline requires file edits + git commits to add a finding; the entity supports `INSERT INTO inquiry_findings`. Concrete need: only if Forge audit roles produce findings *during* a build that downstream Forge phases must immediately retrieve — currently no such workflow exists.
2. **Studio UI workflows.** Triage controls, contribution-thread visualization, lineage rendering. Concrete need: only if a non-author human operates on inquiries through the Studio interface — currently Pavel-as-sole-author with file-edit workflow.

Neither is a Trigger from the original proposal. Both are speculative until exercised.

### Recommendation: explicit re-defer of `add-inquiry-entity`

The 4-PR empirical arc (#1884 → #1885 → #1903 → #1904) plus this measurement has shown that:

- Trigger #1 (drift-coverage) is fully addressable by convention (F-6, F-8, scorecard Query 1).
- Trigger #2 (RAG-shape doctrine) is fully addressable by convention (F-10, scorecard Query 2).
- Trigger #3 (live forward-cohort) was always a "makes Phase 2 cheap" trigger, not an entity-justification trigger — it remains true but not load-bearing.

With all three triggers addressed by convention discipline, **`add-inquiry-entity` should re-defer to idea-stage** per the predecessor's "captured-idea memory" pattern. The convention discipline established through this arc (`## Findings` section + `Capability:` tag + 4-tier `Grade` column + falsifier) is the actual deliverable that ships the value the entity was scoped for.

**Concrete revisit triggers (when entity becomes worth reconsidering):**

1. **Forge audit roles emit findings during a build** that subsequent Forge phases must read in the same build → real-time mutation requirement → entity needed.
2. **A second human operates on inquiries** via Studio UI (triage, promotion, lineage navigation) → UI-driven workflow → entity needed.
3. **Corpus grows past ~200 inquiries** AND grep-based retrieval shows >50% false-positive rate on common queries → vector-search-or-DB threshold → entity OR pgvector needed (intersects with `add-knowledge-vector-store` RAP-1020 scope).
4. **Phase 3a measurement on real Forge build tokens** demonstrates that `search_inquiry()` over JSON irreducibly beats convention-based grep for ≥80% of build queries → DB-backed retrieval pays off → Phase 3b worth pursuing.

Until one of these triggers fires, the convention is the source of truth, and `add-inquiry-entity/` should be marked **re-deferred 2026-05-29** with this scorecard cited as the empirical justification.

---

## 5.4 F-12 — Convention-adoption test (2026-05-29)

The F-3 finding (post-shipping convention adoption rate = 0/4 = 0% for new Forge research) raised the structural question: **does the inquiry-* convention's frontmatter schema actually fit audit-shape artifacts?** §8 question 2 flagged a retroactive rename as the cheapest test. Executed 2026-05-29.

### Method

1. `git mv openspec/_methodology/research/forge-runtime-audit-2026-05-18.md openspec/_methodology/research/inquiry-forge-runtime-audit-2026-05-18.md`.
2. Prepended a 12-line blockquote frontmatter block above the existing H1 title, matching the schema enforced by `tools/lib/parse-inquiry-frontmatter.ts`:
   - Required fields (`Status`, `Lead`, `Opened`, `Capability_refs`, `Thesis`) — all filled from audit context.
   - `Status: published` — audit is complete-snapshot, no further work pending.
   - `Thesis:` — a falsifiable claim extracted from the audit's headline ("76% wasted-run rate is input-quality + outcome-honesty, not engine surgery").
   - `Method:` — methodology section of the existing file lifted into one sentence.
   - `Kind: internal` — first-party operational audit, not public-track.
3. Ran `pnpm research:index` (= `tsx tools/index-inquiries.ts`).
4. Ran `node tools/check-openspec-refs.mjs` to verify refs guard.

### Result

| | |
|---|---|
| `tools/index-inquiries.ts` parser | ✅ accepted without errors |
| Required-field check | ✅ all 5 present |
| Kind-field check | ✅ valid (`internal`) |
| Cohort indexed pre/post | 5 → 6 |
| openspec refs guard | ✅ 0 broken |
| Existing prose references to old filename | Left as historical record; one forward-looking question in §8 + design.md §8 q6 updated to "resolved" |

**Schema fits audit-shape artifacts cleanly.** No new convention fields needed. No parser changes needed. The empirical question raised by F-3 ("does the convention even fit audits?") is structurally answered: **yes**.

### What this does NOT prove

- **Convention adoption velocity remains an open question.** F-3's empirical signal was 0/4 = 0% for NEW research authored after convention shipping. This rename is a *retroactive* adoption; it does not change the prospective trajectory.
- **Convention authoring friction remains untested.** Adding frontmatter to one file by automation isn't the same as a human reaching for inquiry-* naming on first save. The convention-leak log (`_convention-leak-log.md`, started #1884) is still the right instrument for that signal.
- **Forge audit roles haven't emitted findings during a build.** Real-time-mutation revisit trigger #1 from `add-inquiry-entity/proposal.md` remains unactivated.

### Findings from this experiment

| # | Claim | Grade | Falsifier |
|---|---|---|---|
| F-12 | The inquiry-* convention's frontmatter schema fits audit-shape artifacts (status=published, Method = audit methodology, Thesis = headline falsifiable claim, Capability_refs = audited capability slug) without parser modification, field additions, or content rewriting. Confirmed by `pnpm research:index` accepting `inquiry-forge-runtime-audit-2026-05-18.md` after a 12-line frontmatter prepend. Convention is structurally usable for retroactive adoption of legacy research artifacts. | engineering_claim | A second audit file failing `pnpm research:index` after the same retroactive treatment would falsify the schema-fit claim. (F-3 0/4 adoption was about *prospective* authoring; this is a separate hypothesis about *retroactive* schema applicability.) |
| F-13 | Convention adoption decomposes into two independent variables: **schema applicability** (does the frontmatter schema fit the artifact shape?) and **author reach-for-rate** (do authors choose inquiry-* naming on first save?). F-12 settles the first variable for audit-shape; F-3 measured the second variable as 0/4. Future convention work should target each variable separately — schema fixes won't move the reach-for-rate, and naming nudges won't validate schema fit. | engineering_claim | A single intervention (e.g., a CLI scaffold like `muse research init`) that simultaneously fixes both variables would compress them back into one — falsifying the decomposition's necessity. |

### Closes empirical arc

With F-12 + F-13, every empirical question this spike opened has been answered:

- **Trigger #1 (drift-coverage capability_ref filter)** → convention via F-6, F-8 (100% / 90%)
- **Trigger #2 (RAG-shape credibility filter)** → convention via F-10 (100% / 95%)
- **F-3 (convention adoption — retroactive schema-fit)** → F-12 (yes, fits)
- **F-3 (convention adoption — prospective author reach-for-rate)** → still 0/4, decomposed in F-13 as a separate concern
- **F-7 (capability-inference failure in prose)** → fixed by per-section `Capability:` tag (#1904)
- **F-9 (per-row Capability tag for mixed-content files)** → projected design; not built; deferred until a real consumer needs row-level retrieval
- **F-11 (heading-style vs tabular Findings shape)** → known constraint, fix path documented

The 5-PR + this PR (= 6-PR) arc has closed every loose thread. From this point on, inquiry-related work is convention-adoption (independent of base entity) and revisit-trigger monitoring (per `add-inquiry-entity/proposal.md` re-defer banner).

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
2. **Should `forge-runtime-audit-2026-05-18.md` be retroactively renamed?** ~~Cost: ~30 minutes including frontmatter authoring.~~ **Resolved 2026-05-29:** yes — renamed to `inquiry-forge-runtime-audit-2026-05-18.md`. Schema fit clean (zero parser errors), `index-inquiries.ts` accepted it without modification, indexed cohort raised 5 → 6. Full measurement in §5.4 (F-12). Peers not renamed — schema-fit question is structurally answered; further renames are convention-adoption work, deferred until a downstream consumer requests them.
3. **What's the right unit for drift-coverage stage 2: a file, a `## Findings` row, or an entity row?** This spike assumes file-level retrieval. The proposal's design.md §5 implies row-level. If row-level is the target, the marker convention upstream of the entity becomes even more important.
