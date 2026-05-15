# Inquiry workflow gates — AI-as-reviewer pass on v2 (Round 2)

**Owner:** Pavel Rapoport.
**Opened:** 2026-05-15.
**Status:** published.
**Authors:** Claude Opus 4.7 (Anthropic) — AI-reviewer pass in devil's-advocate mode.
**Method:** Adversarial pass over [PR #1168](https://github.com/rapoport-studio/rapoport.studio/pull/1168) — `proposal.md` + `design.md` v2 + `tasks.md`. Rubric carried forward from the 2026-05-14 Round-1 review of v0 (industry-baseline scoring across 15 categories). Each v0 gap re-checked for closure; new gaps surfaced where present.
**Reusable for:** future mandatory-tier Inquiry reviews (this is the first concrete instance of §3.4 dogfooding); calibration baseline for AI-reviewer dismissal-rate falsifier.

> **Methodological disclosure (binding for interpretation):** the same model class (Claude Opus 4.7) authored both v2 and this review. This is a known v2 §3.4 weakness — devil's-advocate via the same model carries confirmation-bias risk. Mitigations applied: (a) explicit adversarial prompting framing throughout this artifact; (b) priority on finding *new* gaps not addressed in Round 1; (c) honest scoring per the original rubric, not self-favouring inflation. **Recommendation for production:** Round-3+ reviews should rotate to a different model class (GPT-5 / Gemini 3 / etc.) to neutralise this concern. Logged as carry-forward gap N-3 below.

---

## Round-1 gap closure verdict

All 11 gaps from the 2026-05-14 Round-1 review re-checked against v2:

| Gap | Round-1 category | Closure | Quality of execution |
|---|---|---|---|
| G1 — solo founder reality | critical | ✅ closed via §3.4 AI-as-reviewer | Strong. Mechanism is concrete (Forge command, contribution emission, gating). Carry-forward N-3 below. |
| G2 — no prior-art citation | critical | ✅ closed via §0 | Strong. Four paradigms cited with real authors. §0.3 names three unique-to-Inquiry properties cleanly. Minor: §0 doesn't cite IEEE/ACM literature on Architecture Knowledge Management — but that's optional polish, not gap. |
| G3 — Atomic Research 4 fields vs Inquiry 3 | critical | ⚠️ closed via push-back + boundary note | Push-back is defensible (epistemic warrant vs actionability), but boundary is *implicit* in v2. New gap N-1 — see below. |
| G4 — author self-categorisation gameable | critical | ✅ closed via §2.0 auto-tier | Strong fix. Heuristics are structural. Override is loud (`<!-- tier-override: -->` in diff). Minor: default rule 7 = `implicit_ok` may be too permissive — see new gap N-2. |
| G5 — arbitrary thresholds | operational | ✅ closed via §5 placeholder marking | Adequate. All numbers explicitly marked placeholder. §5 commits to 90-day calibration. Could be stronger: which dataset, who calibrates, what success criteria — see new gap N-4. |
| G6 — lint too coupled for MVP | operational | ✅ closed via §4 MVP simplification | Strong. Capability intersection deferred to Phase 3. MVP rule simple enough to actually ship. |
| G7 — update existing Inquiry path | operational | ✅ closed via §2.3 cite-existing OR create-new | Strong. Both sub-cases explicit. Sibling change-folder pattern is novel — see new gap N-5 about ordering. |
| G8 — no rollback path | operational | ✅ closed via §5.1 | Adequate. Soft + hard rollback distinguished. Trigger criteria specified (>30% slowdown). |
| G9 — voice consistency on solo founder | minor | ✅ closed partially via §3.1 forward-looking | Right framing. Acknowledges current absence of audience; locks in for future-state. |
| G10 — falsifier verb-detection noisy | minor | ✅ closed via hybrid heuristic | Adequate. ≥50 chars + verb-pattern + blocklist is Goodhart-resistant enough for MVP. May produce false positives on edge-cases (multi-language, terse English) — observable in 90-day calibration. |
| Self-caught: temporal ordering | self-caught | ✅ closed via §2.5 | Strong. Three patterns (pre-PR / parallel / post-hoc) all valid; gating constraint is correct. |

**All 11 closed.** 0 critical regressions. 5 new gaps surfaced (below).

## New gaps surfaced in v2

### N-1 (operational) — Atomic Research 4-field boundary implicit

§2.2 includes the note *"implicit-inquiry covers epistemic warrant only; actionability lives in proposal.md + tasks.md; experiments live in full mandatory-tier Inquiry"*. This closes the G3 push-back conceptually, but the boundary is **not enforced in the lint rule**. An author can legitimately ask: "where does my Experiment-style observation go in implicit tier?". The answer in v2 is buried in §2.2 commentary.

**Fix:** add a `lint_warning` (not error) emitted when implicit-tier evidence field contains words like "we tried", "we tested", "experimented" — pointer back to mandatory tier. Cheap to implement, prevents misclassification.

**Severity:** operational. Deferrable to Phase 2 implementation. Logged in `tasks.md § 2.4`.

### N-2 (operational) — `implicit_ok` as default tier may be too permissive

§2.0 rule 7 makes `implicit_ok` the catch-all default. Practical implication: bulk-renames, doc fixes, dependency bumps with no behavior change all default to implicit-tier, requiring thesis + evidence + falsifier. This is **overhead for trivial PRs**.

**Counterpoint:** §2.0 rule 5 catches operational/bug. Anything not matching rules 1-6 plausibly has *some* decision context worth a 5-line implicit. The Goodhart risk is the *other* direction — authors mass-overriding to none.

**Recommendation:** ship v2 as-is; track the override-rate falsifier in §5 over 90 days. If too many `tier-override: none` events, refine rule 7. If too few, the default works.

**Severity:** operational. Phase 4 calibration tunes this.

### N-3 (critical) — §3.4 same-model reviewer not addressed in design

§3.4 says *"AI agent (Claude Opus, GPT-5, or future model) is prompted with the Inquiry artifact"*. It allows the same-model case (Pavel running Claude Opus to review a Claude-Opus-co-authored Inquiry). This artifact you're reading proves the methodological concern is real — the same model class does NOT produce maximum adversarial signal.

**Fix:** add §3.4.1 — "model rotation requirement". For mandatory-tier `kind = public` Inquiries, the reviewer model MUST differ from the model that contributed >50% of the Inquiry's Contributions. For `kind = internal`, recommended but not required. Tracking implementation: `Inquiry.contributions[].author_id` is already model-tagged (`agent:claude:opus-4-7`), so the rule is queryable.

**Severity:** critical for v2.x; v2 ships as-is with the disclosure-via-this-artifact, then v3 adds §3.4.1.

### N-4 (minor) — Calibration process under-specified

§5 commits to 90-day calibration but says nothing about: who runs it (Pavel solo? AI agent? quarterly review?); what dataset (every PR? sampled?); what's the bar to update a threshold ("revise the spec" — but how exactly?). Without process, "calibrate at 90 days" becomes a wish.

**Fix:** add §5.2 "Calibration process":
- Trigger: 90 days after Phase 2 cut-over date (recorded in archive metadata)
- Data: ALL PRs touching `openspec/changes/` since cut-over, aggregated automatically via a `tools/measure-research-lineage.mjs` script (Phase 4 task)
- Decision: owner reviews the dataset + emits a new change folder `tune-inquiry-workflow-gates-thresholds-<date>` if any threshold updates
- No-update default: thresholds stay as v2-locked; absence of decision = stable

**Severity:** minor. Phase 4 task. Logged in `tasks.md`.

### N-5 (minor) — Create-new sub-case ordering ambiguous

§2.3 sub-case (b) "Create-new" says: *"Both PRs progress in parallel, the Inquiry merges first"*. Operational question: what if the consuming PR is approved but the sibling Inquiry PR isn't yet? Does the consuming PR wait? Can they merge atomically? What if a third PR depends on the Inquiry while it's mid-review?

**Fix:** specify in §2.3 that consuming PR's CI lint passes only when sibling Inquiry slug resolves (i.e. the Inquiry change folder exists in `openspec/changes/` OR `openspec/inquiries/<slug>/` if the entity exists). Merge ordering: the sibling Inquiry change merges first, then consuming PR triggers a re-run of CI lint and merges. No "atomic merge" required.

**Severity:** minor. Adds 4 lines to §2.3.

## Score

Rubric carried forward from Round-1 (15 categories, 1-10 each):

| Category | Round 1 | Round 2 | Δ | Note |
|---|---:|---:|---:|---|
| Strategic framing | 9/10 | 9/10 | — | Compound-asset thesis retained |
| Three-tier categorisation | 8/10 | 9/10 | +1 | §2.0 auto-tier added |
| Falsifier-as-discipline | 9/10 | 9/10 | — | Hybrid heuristic in §2.2 |
| Boundary clarity | 8/10 | 9/10 | +1 | §3.4 closes the solo-founder hole |
| Phase sequencing & gating | 8/10 | 8/10 | — | Unchanged |
| Open question defaults | 8/10 | 9/10 | +1 | §6 defaults explicit and minimal |
| Solo founder context realism | 4/10 | 8/10 | +4 | §3.4 directly addresses |
| Reviewer-leverage realism | 4/10 | 8/10 | +4 | §3.4 mechanism gives reviewer signal |
| Implicit-Inquiry tier completeness | 6/10 | 8/10 | +2 | Epistemic-warrant boundary noted (N-1 carry) |
| Quantitative metric grounding | 5/10 | 7/10 | +2 | Placeholders honest; N-4 carry on process |
| Industry prior-art citation | 4/10 | 9/10 | +5 | §0 closes G2 strongly |
| Mandatory tier discoverability | 6/10 | 9/10 | +3 | Auto-tier closes G4 |
| Lint rule pragmatism | 6/10 | 9/10 | +3 | MVP simplification correct |
| Rollback path | 3/10 | 8/10 | +5 | §5.1 explicit |
| AI agent role in research | 5/10 | 8/10 | +3 | §3.4 + grant system integration |

**Round 1 average:** 6.2 (rounded to 6.5).
**Round 2 average:** 8.5.

**Score: 8.5/10.** Meets the Phase 1 acceptance target.

## Remaining gaps (priority for v3 or Phase 2 task list)

1. **N-3 (critical-for-v2.x):** add §3.4.1 model rotation requirement. Mandatory for `kind=public`, recommended for `kind=internal`. Logged in `tasks.md` Phase 2.
2. **N-1 (operational):** add lint warning for experimental language in implicit tier. Phase 2 task.
3. **N-4 (minor):** add §5.2 calibration process. Phase 4 task — acceptable to defer.
4. **N-5 (minor):** specify create-new ordering in §2.3 (4-line addition). Can land in this PR if owner agrees, or Phase 2 task.

## Decision candidate

**Ratify with carry-forward fixes.** Score 8.5/10 meets the Phase 1 acceptance target. Five new gaps are either (a) operational and deferrable, or (b) require external dependency (model rotation per N-3 needs a second model available in production). v2 is good enough to merge as the foundation; N-3 should ship as a fast follow-up change folder before Phase 2 cut-over since it's a real adversarial-bias hole this very review artifact demonstrates.

**PR #1168 recommendation:** move from draft → ready-for-review. Pavel sign-off completes RAP-840 and Phase 1 of RAP-835.

**Carry-forward Inquiry items:** once `add-inquiry-entity` ships (RAP-842 Phase 3), file this artifact as an `InquirySource` (kind: `internal_artifact`, credibility: `experimental`) on the parent Inquiry for `add-inquiry-workflow-gates`. The N-1 through N-5 items become `InquiryContribution` rows with `kind = objection`.
