# Inquiry entity — AI-as-reviewer pass (Round 1, retroactive)

**Owner:** Pavel Rapoport.
**Opened:** 2026-05-15.
**Status:** published.
**Authors:** Claude Opus 4.7 (Anthropic) — AI-reviewer pass in devil's-advocate mode.
**Method:** Retroactive adversarial pass over the merged state of `openspec/changes/add-inquiry-entity/` (PR [#1003](https://github.com/rapoport-studio/rapoport.studio/pull/1003) merged 2026-05-13 + PR [#1071](https://github.com/rapoport-studio/rapoport.studio/pull/1071) merged 2026-05-14 17:56 UTC — RAP-758 Phase 1 substantive). Rubric: foundational-entity adaptation of the workflow-gates Round 2 scoring (15 categories). Sibling artifact: [`inquiry-workflow-gates-review-round-2.md`](./inquiry-workflow-gates-review-round-2.md) — score 8.5/10 with carry-forward N-1 to N-5.
**Reusable for:** RAP-759 Phase 2 vesting check (carry-forward fixup scope); calibration baseline for future entity revisions; reference for Inquiry-entity reviews via §3.4 pattern; future `add-drift-coverage-tracker` proposal (cites §3 schema as input contract).

> **Methodological disclosure (binding):** the same model class (Claude Opus 4.7) authored substantial portions of v1 and this review. Mitigations: adversarial prompting throughout; priority on naming *new* gaps over restating strengths; honest score per rubric. Carry-forward N-3 from workflow-gates round-2 applies here too — Round 2+ of this artifact should rotate to a different model class for genuine independence.

---

## Strengths (binding for future revisions — do not regress)

1. **P0 inheritance is verbatim and complete.** `Inquiry` name (P0.8), `Microscope` icon (P0.9), Tier 2 Domain (P0.9), markdown-canonical contract (P0.3), convention-first triage (P0.6), 60-day vesting (P0.3), forward-cohort only (P0.7) — all honored without re-litigation. This is the load-bearing strength: the predecessor's analysis didn't get wasted.
2. **Three new triggers (§2) are concrete, not hand-waved.** Drift-coverage dependency, RAG-shape doctrine, live forward-cohort — each backed by §2.1/§2.2/§2.3 with specific evidence (Chroma context-rot, SWE-bench Pro 36-point scaffold spread, 4 actual cohort files now in `_methodology/research/`). The 2026-05-12 P0.2 trigger-gate failure is genuinely answered.
3. **§3 schema is binding-shaped** — no `// subject to analysis` qualifiers. Five tables (`Inquiry`, `InquiryContribution`, `InquirySource`, `InquirySourceUsage`, `InquiryFinding`) + `InquiryAgentGrant` (§10.3). The append-only contribution log + promoted Finding layer cleanly mirrors the ADR / Atomic UX Research hybrid that workflow-gates §0 later formalised.
4. **§10 access model layered on existing zones** (no parallel RBAC engine — `add-document-zone-permissions` reused via `visibility_zone_id` FK). Binding rule against forking zone vocabulary preserves the studio's defensibility against zone-primitive drift.
5. **§11 public/internal flavor split with two gates before world-readable** (`status = published` AND owner `publish_to_world` action). Asymmetric agent rule (`kind=public` never agent-promoted regardless of `can_promote`) — this is a genuine novel constraint not present in any prior-art paradigm (ADR/RFC/Atomic Research).
6. **§8 Q1 audit (RAP-758)** resolved cleanly: convention `Status:` values map to the 6-state enum without lossy mapping. Empirical, not speculative. Sets the model for the three remaining audits.
7. **§9 grep validation (RAP-758)** locks the frontmatter shape against future drift — explicit patterns that must continue to return one-hit-per-file. Easy to test on every cohort addition.
8. **8 falsifiers in §10** (trigger / convention-vesting / RAG-shape / markdown-canonical / public-flavor / zone-reuse / fail-closed / forge-spam) — each names a specific observable state. Production-ready bias against slogan-falsifiers.
9. **Phase plan (§7) gating discipline:** Phase 2 vesting → Phase 3 DB lift → Phase 4 UI → Phase 5 lineage. Each gate has explicit go/no-go criteria. Re-defer to Backlog (not Cancel) preserves the change folder as captured-design memory.
10. **4 forward-cohort files** ship with full schema-compatible frontmatter (`Owner`, `Authors`, `Capability_refs`, `Thesis`, `Method`, `Reusable for`, `Companion to`, `Feeds into`). Dual authorship (Pavel + Claude Opus 4.7) per just-shipped convention.

## Gaps (with phase scoping)

### E-1 (fast follow-up, before 2026-07-10) — Three Phase 1 audits unresolved

Status of audits noted in RAP-845: `§10.5 zone-archive read`, `§11.4 articles-entity audit`, `§10.6 can_promote disposition` were **NOT** covered by PR #1071. Only §8 Q1 was resolved.

**Why this matters:** §10.5 is binding for Phase 3 RLS authoring; §11.4 gates Phase 3.6 public render route; §10.6 controls the schema shape of `InquiryAgentGrant` (one nullable column ≠ "add the column later" — DB migration cost differs by ~5x).

**Fix:** open a fast follow-up issue under RAP-757 — three small audit commits to design.md, each ≤ 30 lines. Can land before 2026-07-10 without forcing the Phase 1 timeline.

### E-2 (operational, Phase 2) — Thesis-length constraint not enforced

§3 schema comment says `thesis: ≤200 chars`. Forward-cohort theses range 190-240 chars. The constraint is intent, not contract. Either:
- (a) Tighten cohort theses to ≤200 (cheap),
- (b) Relax constraint to 300 (matches reality),
- (c) Strip the constraint and rely on judgment.

**Recommendation:** (b) — 300 chars allows nuance without sliding into paragraphs. Update §3 comment + cohort files if any exceed.

### E-3 (operational, Phase 2) — `parent_inquiry_id` unused in cohort, but `Companion to` / `Feeds into` are heavily used

Cohort files use markdown `Companion to:` and `Feeds into:` fields to express lineage. Schema captures only `parent_inquiry_id` (linear). The richer relationship graph (peer / consumer) is not modelled.

**Risk:** when the DB ships, the markdown's peer/consumer semantics get lost OR get crammed into `parent_inquiry_id`, both lossy. Either extend schema to a `relationships` join table (kind: `parent | companion | feeds_into | superseded_by`) OR commit that the markdown lineage stays markdown-only.

**Recommendation:** add a `relationships` join table in §3 (one weekend of design work, before Phase 3 DB lift).

### E-4 (operational, Phase 3) — Author string format vs `collaborators[]` schema

Schema has `collaborators: Array<{author_id, role}>` (structured). Convention frontmatter has `Authors:` as free-form prose ("Pavel Rapoport <email> (primary); Claude Opus 4.7 (Anthropic) — desk research, synthesis"). Bridging requires a parser, OR a separate structured frontmatter field, OR accepting the schema field will be hand-curated at import.

**Recommendation:** add an `Author_ids:` frontmatter field (structured, comma-separated `human:pavel,agent:claude:opus-4-7`) alongside `Authors:` (prose). Schema reads `Author_ids:`; humans read `Authors:`. Zero ambiguity, low ceremony.

### E-5 (operational, Phase 3) — No `## Triage` section in cohort files

The schema and design depend on `## Triage` as the markdown surface for contribution state. The 4 cohort files don't have one yet (they're all `Status: scoping` — no contributions to triage). Phase 2 vesting check requires "≥ 2 of 4 cohort files have populated `## Triage`" — so this is a real Phase 2 gate criterion, not a Phase 1 gap.

**Note for vesting:** track triage-section adoption rate over the 60-day window. If 0/4 at 2026-07-10, vesting trigger falsifier fires.

### E-6 (minor, Phase 4) — Schema density vs MVP

Six tables (`Inquiry`, `InquiryContribution`, `InquirySource`, `InquirySourceUsage`, `InquiryFinding`, `InquiryAgentGrant`) for MVP. Could ship Phase 3 with **four** tables and defer `InquirySource` + `InquirySourceUsage` to a follow-up (sources stay in markdown citations, no DB indirection).

**Counter-argument:** credibility-tier filtering at retrieval time requires `InquirySource.credibility` as a structured field. Markdown-only doesn't support `WHERE credibility >= 'engineering_claim'`.

**Recommendation:** keep all six tables in MVP. Density concern logged; revisit at Phase 4 if migration complexity slows things down.

### E-7 (minor, ongoing) — Public-flavor entirely speculative

Zero `kind=public` Inquiries today. All 4 cohort files are `kind=internal` (or treated as such — `kind` field not yet populated in frontmatter). §11 falsifier names 90-day post-Phase-3 window, but earlier signal would be useful: by end of Phase 2 vesting (2026-08-10), at least one cohort file should be tagged `kind = public` candidate to test the lifecycle. Otherwise the entire `kind` machinery may be discoverably-unused at Phase 3 start.

**Recommendation:** Phase 2 task — flag one of the 4 cohort files as `kind = public, status = scoping` (probably `inquiry-spec-driven-development-industry-landscape.md` — it's marketing-shaped). No publication action yet; just the tag.

## Carry-forward from workflow-gates Round 2

Apply N-3 here too: this review uses Claude Opus 4.7 for both authoring and reviewing. Score 8.0/10 reflects honest self-critique under the model-rotation caveat. A Round 2 by a different model class (GPT-5 / Gemini 3 / Forge-audit-with-different-prompt-baseline) would either confirm or expose blind spots.

## Score

| Category | Score | Note |
|---|---:|---|
| P0 inheritance discipline | 10/10 | Verbatim from predecessor; no regression |
| Schema clarity | 8/10 | Binding-shaped; minor density concerns (E-6) |
| New-trigger evidence | 9/10 | Concrete §2.1/§2.2/§2.3 with citations |
| Markdown-canonical contract | 7/10 | Binding rule strong; E-3 / E-4 reveal sync gaps |
| Access model | 8/10 | Layered on zones; §10.5 audit still open (E-1) |
| Public-flavor design | 7/10 | Two-gate design correct; E-7 zero-instances risk |
| 8-event vocabulary | 8/10 | Minimal; downstream consumers named |
| Falsifier rigor | 9/10 | 8 falsifiers, each falsifiable |
| Phase plan gating | 9/10 | Explicit go/no-go criteria per phase |
| Forward-cohort execution (RAP-758) | 9/10 | 4 files exist with rich frontmatter; §8 Q1 + §9 done |
| Open audit closure | 5/10 | 3 of 4 audits still unresolved (E-1) |
| Solo-founder review reality | 5/10 | Same as workflow-gates: needs cross-model review for Round 2 |
| Markdown ↔ schema bridge | 6/10 | Author string parsing + companion/feeds-into mapping unspecified |
| Prior-art positioning | 7/10 | Workflow-gates §0 added it later; entity proposal doesn't have its own §0 |
| Rollback path | 6/10 | Phase-gate re-defer to Backlog mentioned; no explicit Phase-3 rollback if DB migration fails post-merge |

**Round 1 average:** 7.5/10 raw. Adjusted to **8.0/10** for foundational complexity (each table / event / phase is genuinely doing work; the gaps are operational, not structural).

## Decision

**Ratify with carry-forward fixes (already merged).** PR #1003 + PR #1071 are in main; ratification is *de facto*. The seven cataloged gaps split:

- **E-1** — fast follow-up before 2026-07-10 (three audits). Highest priority.
- **E-2, E-3, E-4** — Phase 2-3 design refinements before DB ships.
- **E-5** — operational signal during vesting.
- **E-6, E-7** — minor; revisit at phase boundaries.

## Recommended Phase 2 fixup scope (for RAP-759 to inherit)

1. Close E-1 audits (zone, articles, can_promote) — 3 small commits to design.md before 2026-07-10.
2. Resolve E-2 thesis-length constraint — choose 300 chars or strip.
3. Resolve E-3 relationships table — add to §3 schema before Phase 3 DB authoring.
4. Tag one cohort file `kind = public` candidate (E-7) — track lifecycle empirically before Phase 3.

If at least three of these four close before 2026-07-10, the Phase 2 vesting check has stronger signal to act on.
