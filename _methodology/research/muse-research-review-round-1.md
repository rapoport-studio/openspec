# Muse research mode — AI-as-reviewer pass (Round 1)

**Owner:** Pavel Rapoport.
**Opened:** 2026-05-15.
**Status:** published.
**Authors:** Claude Opus 4.7 (Anthropic) — AI-reviewer pass in devil's-advocate mode.
**Method:** Adversarial pass over `openspec/changes/add-muse-research-mode/` (proposal.md + design.md + tasks.md) — [PR #1220](https://github.com/rapoport-studio/rapoport.studio/pull/1220). Rubric adapted from the workflow-gates Round 2 scoring (15 categories). Special focus: does the proposal correctly *apply* the workflow-gates discipline to itself, since it is the first non-trivial change to do so?
**Reusable for:** future Muse subcommand proposals (research / synthesise / review pattern); third concrete instance of §3.4 dogfooding (after RAP-840 round-2 and RAP-845 round-1) — calibrates the AI-reviewer mechanism on heterogeneous proposal shapes.

> **Methodological disclosure (binding):** the same model class (Claude Opus 4.7) authored both the proposal and this review. Mitigations: adversarial prompting, priority on new gaps, honest scoring. Carry-forward N-3 from RAP-840 round-2 applies — Round 2+ of this artifact should rotate to a different model class. The proposal itself enforces this rule (§3.4.1 model rotation) for `kind = public` Inquiries; this review is `kind = internal` and not yet bound by it.

---

## Workflow-gates self-application — verdict

The proposal is **the first non-trivial change to apply workflow-gates discipline to itself**. Specific compliance check:

| Discipline element | Required for `new_capability`? | Present? | Quality |
|---|---|---|---|
| `## Research lineage` section in proposal.md | ✓ | ✓ | High — fully-formed §2.3 mandatory-tier shape |
| Cite existing Inquiry OR create-new sibling | cite-existing chosen | ✓ | `inquiry-context-as-bottleneck` is genuinely the relevant Inquiry; not a forced citation |
| ≥ 1 cited Finding with falsifier | ✓ | ✓ | Two findings cited with their original falsifiers verbatim |
| Falsifiers in proposal tie back to cited Inquiry | not required | ✓ | **Scaffold-thesis falsifier** ties back to cited Inquiry's load-bearing claim. Popperian recursion is rare. **Strong.** |
| Categorisation declared | ✓ | ✓ | `new_capability` → Mandatory; correctly auto-tiered per §2.0 |
| Phase plan with gates | recommended | ✓ | 5 phases each with explicit gate criteria |
| Falsifiers (binding gates) | recommended | ✓ | 4 falsifiers, each falsifiable |
| Alternatives considered | recommended | ✓ | Five alternatives (A-E) in design.md §1; four explicitly rejected with reasons |

**Compliance score: 9.5/10.** This is the cleanest workflow-gates application produced so far in the inquiry-track corpus.

## Strengths

1. **Self-applicational integrity.** The proposal applies its enabling framework's discipline to itself — including Popperian recursion (its own falsifier ties to the cited Inquiry's falsifier, so a defeat of either premise defeats both). This is a rare and durable property.
2. **Alternative space honestly explored.** Five options enumerated; four rejected with named reasons (semantic mismatch / cost / premature / no demand signal). Reviewer cannot reasonably argue "you didn't consider X" — X is probably covered.
3. **Cost gate in CLI.** `--max-cost-usd 5` default + pre-flight estimate prevents runaway spend. Cost falsifier (per-run > $5 sustained → reduce defaults) explicitly named.
4. **§3.4.1 model rotation enforced at CLI**, not just spec. design.md §7 has implementation sketch. Operational note about solo-founder API-key constraint matches workflow-gates §3.4.1 binding rule.
5. **Cite-existing path correctly chosen.** Could have over-engineered by creating a sibling Inquiry; instead cites already-shipped Inquiry. Demonstrates the cite-existing case from §2.3 in practice.
6. **Five open questions with default decisions.** Each question has a default. Reviewer can confirm or override; ratification doesn't stall on unresolved Q&A.
7. **Phase 4 honest about gating.** Doesn't pretend to do entity-DB integration in MVP; defers cleanly to `add-inquiry-entity` Phase 3 exit (2026-07-10+).
8. **Pre-Phase-3 markdown-only output design.** Reuses the same convention shape as forward-cohort files — discipline is consistent.

## Gaps (numbered for follow-up)

### Critical (close before Phase 2 starts)

**P-1: Streams hardcoded to 4, no auto-detection.** Not every research thesis benefits from all four canonical framings. A purely-architectural research wastes tokens running the strategy + decision-history streams. **Fix:** add `--framings` flag in Phase 2 to opt-in to subset; default stays 4 for MVP but unlock at Phase 3.

**P-2: Synthesis pass recursive context-rot risk.** Synthesis sub-agent receives **all 4 stream outputs simultaneously**. Each stream output could be ~3K tokens; 4 × 3K = 12K input to synthesis. Acceptable, but the cited Inquiry itself warns about Chroma context-rot at 50K+. If `--depth deep` is used (~50 sources per stream → ~10K per stream output), synthesis input balloons to 40K — entering the context-rot danger zone. **Fix:** cap synthesis input at 30K tokens; if exceeded, do hierarchical synthesis (2 + 2 + final) instead of flat. Document in design.md §4.

**P-3: Solo-founder availability dependency for `--kind public`.** §3.4.1 mandates model rotation for `kind = public`. With one API key at MVP, `--kind public` is unusable. Proposal acknowledges this in §7 but the CLI behavior when only one model class is configured is undefined. **Fix:** `muse research scope --kind public` should fail at CLI-config-check time with clear error ("Configure a second model class via env $MUSE_REVIEWER_MODEL_CLASS before scoping public Inquiries").

### Operational (Phase 2 implementation concerns)

**P-4: Cost-estimate model-specific.** Pre-flight assumes Claude Opus 4.7 pricing. If user runs `--model anthropic:claude-sonnet-4-5`, estimate is wrong. Need model-aware pricing table.

**P-5: No retry/resume on stream failure.** If stream 3 fails after stream 1+2 completed at $1.20, restart costs another $1.20+. **Fix:** persist per-stream completion state in `inquiry-<slug>.md` `## Triage` section; resume failed streams from `run-stream <slug> <stream-id>`.

**P-6: No human-in-loop checkpoint.** All 4 streams run in parallel without owner intervention. If stream 1 output shows obvious bias / hallucination, owner can't course-correct without re-running everything. **Fix:** Phase 3 — add `--interactive` flag that pauses after each stream for owner triage.

**P-7: Source recency not enforced in prompt.** WebSearch results vary by index date. Stream-2 (strategy) lookups can return outdated competitive landscape. **Fix:** prompt template should require sub-agent to flag sources older than 12 months as `stale: true` in citation metadata.

### Minor

**P-8: Linear-issue creation behavior on `scope` without `--linear-parent`.** design.md §10 Q2 lists "default position: scope creates immediately" but doesn't specify what happens if env config `MUSE_RESEARCH_LINEAR_EPIC` is unset. Phase 2 implementation will need to define.

**P-9: No tests for the 4-canonical-framing prompt invariants.** tasks.md §2.7 lists a model-rotation test but no test that asserts each framing-prompt actually produces output shaped like its claimed framing (i.e. metric stream should produce metric-shaped Findings, not architecture-shaped).

**P-10: `--kind public` semantics at scope time.** Workflow-gates §11.3 says agents NEVER scope public, but Pavel runs `muse research` as human invoker — does that count? Implied yes (it's the founder, not an agent). But the CLI should make this explicit (`Inquiry.kind=public` requires `--lead human:*` not `agent:*`).

## Score

Rubric adapted to a Phase-1 spec proposal:

| Category | Score | Note |
|---|---|---|
| Workflow-gates self-application | 9/10 | Cleanest yet; sets template for future proposals |
| Research-lineage citation quality | 9/10 | Cite-existing chosen well; falsifier recursion is rare |
| Alternative space exploration | 9/10 | 5 alternatives, 4 rejected with reasons |
| Phase plan gating discipline | 8/10 | 5 phases, clean gates; minor over-segmentation possible |
| Falsifier rigor | 8/10 | 4 falsifiers, all named and observable |
| Open-question coverage | 8/10 | 5 questions with defaults; could have anticipated P-3 |
| Cost discipline | 8/10 | --max-cost-usd is good; model-aware estimate (P-4) missing |
| Implementation sketch realism | 7/10 | reuse strategy clear; retry/resume (P-5) and interactive mode (P-6) missing |
| Solo-founder context realism | 6/10 | acknowledges single-key constraint but doesn't enforce at CLI (P-3) |
| Markdown ↔ schema bridge | 8/10 | Output contract §5 maps to add-inquiry-entity convention |
| Pre-existing-Inquiry leverage | 9/10 | cites real Inquiry; doesn't manufacture one |
| Prompt-template specification | 7/10 | 4 framings named but invariants per framing not formally specified (P-9) |
| Public-flavor handling | 6/10 | §3.4.1 enforcement design is good; scope-time CLI behaviour underspecified (P-3, P-10) |
| Recursive context-rot awareness | 6/10 | Cited Inquiry warns about context-rot; synthesis pass risks it (P-2) |
| Operational reuse from `forge/core` | 9/10 | Carve-out invocation correct; no boundary violations |

**Score: 8.0/10.** Meets Phase 1 acceptance target. Comparable to entity round-1 (8.0); below workflow-gates round-2 (8.5) due to less iteration time, and 3 critical Phase-2-blocking gaps (P-1, P-2, P-3).

## Decision

**Ratify with critical fixes before Phase 2 starts.**

- **Block Phase 2 kickoff on P-1, P-2, P-3 closure** (small design.md amendments — ~25 lines total). All three are Phase-1 design refinements, not implementation work. Can land in a fast follow-up commit on this same branch before merge, OR as a separate follow-up PR before Phase 2 sub-issue is opened.
- **P-4 through P-7** belong to Phase 2 implementation; track as Phase 2 sub-issues.
- **P-8 through P-10** are minor; can be deferred to Phase 2 verification gate.

Either path (fast follow-up commit OR follow-up PR) honors workflow-gates discipline — the critical fixes ARE the iteration this Round 1 review was supposed to produce.

**Recommendation:** since the PR is fresh and reviewer-attention is high, **fast follow-up commit on this branch** to close P-1 + P-2 + P-3. Score after fix expectation: ≥ 8.5/10.

## Carry-forward for follow-up reviews

- N-3 from RAP-840 (same-model reviewer concern) applies here too — Round 2+ should rotate model class.
- Apply this review's gaps (P-1 through P-10) as Phase 2 task ledger; track closure rate as `muse research review` calibration data.
- If `muse research` ships and is used 5+ times, run Round 2 of this review using a different model class — empirical baseline for §3.4.1 rotation benefit.