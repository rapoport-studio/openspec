# Migration Track

> OpenSpec is the Source of Truth of our platform.
>
> What you write in it becomes our shared reality — and we will
> build from it.
>
> We do not translate your words into our terms. We help you say
> what you mean, in a language that both you and your future
> partners can stand on.
>
> This is how we speak to those who come to us.
> This is how we invite founders to dream — and to create what
> they have not yet seen.

> **Status:** draft v0.1 — methodology layer. Pilot run #1: vivod (Linear epic
> VVD-1). Engagement framing: see `rapoport-studio-engagement.md`
> § Engagement tracks. The internal companion to this document —
> the thesis and operating posture behind the methodology — is held
> separately and is not part of this public corpus.

---

## 1. What this is

A founder arrives at the studio with a product they have built and no longer
fully recognise. They can list features but not name them structurally. They
know what users do but not how the system represents it. They speak about their
product in three different languages depending on who is asking: technical to
engineers, commercial to investors, operational to the team — and the gap
between these three is widening every month.

Migration Track is the studio's response. It is a guided transition into a new
working environment, where one language replaces the three: **OpenSpec, used as
the Source of Truth of the product.**

In this environment, structure is not hidden in code that only engineers read.
It is not buried in pitch decks that the team has never seen. It is written,
once, in language that the founder can hold in their head — and that engineers,
partners, designers, investors can all read directly. The translation layers
between dialects collapse. The product becomes legible to itself.

This is what Migration Track delivers — not a methodology, not a course, not a
deliverable. **An environment shift.** The founder crosses from
operating-by-instinct, where every conversation re-explains the same product in
a different tongue, into operating-through-shared-reality, where what is written
is what we build.

The work has two outputs, inseparable:

1. **The founder, now fluent in this environment** — able to describe their own
   product structurally, to reason about modules and their boundaries, to invite
   partners into specific parts rather than guarding the whole.
2. **A reconstructed OpenSpec corpus in studio dialect** — the trail the founder
   leaves behind in walking the transition. The corpus has independent value as
   documentation, but it exists only because reconstruction is the method
   through which fluency forms.

Migration Track is one of the studio's defined engagement tracks (see
`rapoport-studio-engagement.md` § Engagement tracks). It runs after Discovery,
inside the engagement frame established by charter and kickoff. The methodology
layer (this document) describes what happens between P0 and P6. The engagement
layer describes how the studio enters and exits the relationship around it. They
are different scopes; this document does not replace any existing engagement
artifact.

A note on what reconstruction means here. The founder's product likely already
has an OpenSpec corpus — formal, AI-generated, with numbered requirements. That
corpus is one of the source materials for reconstruction, alongside the codebase
and the founder's lived operational knowledge. Migration Track does not throw it
away. It treats it as a starting reading, to be questioned, reconciled, and
rewritten in studio dialect — owned by the founder, not inherited from the
machine that produced it.

## 2. What this asks of you

A transition is not a course. It is a temporary shift in how you live with your
product, and it works only if you enter it deliberately. Before starting, the
founder agrees to four things.

### 2.1 Presence

Four to six hours per week, across eight to twelve weeks, in sessions of roughly
ninety minutes. This is foreground attention, not background work between calls.
The methodology is designed around your active thinking; without it, the gates
do not catch anything and the corpus becomes performance.

### 2.2 Honesty about not knowing

The hardest moment in Migration Track is the one where you realise you do not
understand a part of your own product. This moment happens. It is the point at
which structural thinking starts to actually form. The methodology only works if
you name the confusion when you feel it, rather than producing a plausible
answer to move forward.

The comprehension gates (§7) are designed to detect plausibility. If you
optimise for passing them — answering what sounds right rather than what you
mean — you complete the track having practised theatre, not transition. The
studio cannot protect you from this. Only your honesty can.

### 2.3 Independence at the end

The track is finite. Somewhere around week ten, you should begin to feel that
Muse is slowing you down rather than helping you forward — that you can answer
the structural questions before they are asked, that you can read the next
entity without needing the previous one explained. **This is the signal of
completion, not of failure.** The studio will name it explicitly when we see it,
and step back.

If, at the end of the track, you still need the studio to think structurally
about your product, the methodology has failed. Our success is measured by the
moment we become unnecessary to you.

### 2.4 Discomfort tolerance

Phases P2 and P3 produce a recognisable crisis: *I built this, and I do not
understand it.* This is uncomfortable. It is also the most productive moment of
the entire run, because it is when the new environment actually opens. The
studio will name this moment when it arrives, will not minimise it, and will not
rush you past it with reassurance. We will stay with you at the threshold long
enough to cross.

If you want a track that protects you from this moment, Migration Track is the
wrong product. There are excellent tools that will build software for you
without asking you to look at it. We are not one of them.

## 3. What this is not, and does not deliver

Stated clearly because the methodology sits between several established
practices and is often confused with each.

**Not a bulk spec reverse-engineering.** Bulk AI-generated specs from code look
plausible on the surface but diverge from reality (see
`openspec/_methodology/research/repo-migration-tooling-2026.md` for documented
spec-inference failure modes). Migration Track is not that. Reconstruction here
is conducted *by* the founder, slowly, with gates, and the artifact's primary
value is the act of producing it, not its textual quality.

**Not EventStorming or a workshop.** EventStorming is synchronous, group-based,
sticky-note-driven, and discovery-oriented. Migration Track is asynchronous, one
or two learners, chat-driven, and reconstruction-oriented. They are complements,
not alternatives.

**Not a single-change onboarding walkthrough.** Learning the OpenSpec lifecycle
by walking through one small change teaches the mechanics. Migration Track
teaches structural domain thinking by walking through one whole product —
different scope, different time, different goal.

And, as honest expectation-setting, what the track does **not** deliver — derived
from the well-known reliability cliff in AI-assisted builds, where the last
fraction of a build is where most failures concentrate:

- **Not a deployable product.** The track produces a reconstructed as-is spec, a
  migration plan, and the founder's understanding. Building the migrated product
  itself is a separate engagement (or done by the founder's own team).
- **Not exhaustive coverage.** Some entities, edge cases, and dead-code branches
  are excluded by scope decisions in P0 and P6. The spec is honest about its
  boundaries.
- **Not a guarantee of code quality.** The spec captures intent; tests and
  reviews capture quality. They are layered, not interchangeable.

If the founder needs an end-to-end "describe it and ship it" experience,
Migration Track is the wrong product — other tools serve that need. Migration
Track is for founders who want to **own** their architecture mid-migration, not
bypass it.

## 4. Roles

- **Learners.** The founder and any co-builders. Maximum two per run for the
  comprehension gate (§7) to function. Pavel + Misha is the canonical pair. Both
  learners pass through every cluster from opposite sides (author / reviewer),
  rotating, so understanding is shared.

- **Muse — reviewer.** AI in reviewer mode (a distinct mode from Architect — see
  §10). Muse holds the existing code and the legacy spec as searchable context,
  runs review passes after each phase, challenges as devil's advocate, and runs
  comprehension gates. Muse never authors the new spec — learners do, Muse
  checks.

- **Pavel — closing authority.** Muse posts gate verdicts (`pass / partial /
  fail`) into the run's Linear epic as comments. Pavel closes sub-issues based on
  those verdicts. This deliberately keeps a human in the loop on the most
  critical methodology decision — does the learner actually understand? At
  scale, this role transfers to senior studio collaborators.

## 5. The phases

Bottom-up by design, with explicit preflight and handoff.

| Phase | Topic | Artifact | Time bound |
|---|---|---|---|
| **P0** | Preflight | Cost cap, tooling check, validation harness, scope decision, golden set | ≤ 1 session |
| **P1** | OpenSpec literacy | Vocabulary mapping; learner can explain in plain language | ≤ 2 sessions |
| **P2** | Domain model | `domain-modeling.md` for this product | ≤ 3 sessions |
| **P3** | Entities (clustered) | One Entity Dossier per cluster → spec in `openspec/specs/` | Variable by cluster count |
| **P4** | Relationships | One ER-level spec | ≤ 4 sessions |
| **P5** | Application surfaces | Screen specs grounded in entities + relationships | Variable by surface count |
| **P6** | Target deltas & migration plan | Migration plan + handoff package | ≤ 4 sessions |

Each phase is one Linear sub-issue. P3 splits into one sub-issue per entity
cluster. A phase closes only when both gates pass (§7).

The bottom-up order is non-negotiable: Muse refuses to work on phase N while
N−1 is open. This is the only way to prevent learners from leaping to the
"interesting" surface phases (P5, P6) without the structural grounding to
discuss them honestly.

### 5.1 Mid-run checkpoint (P3 → P4)

After P3 closes and before P4 starts, the run pauses for a 45-minute structured
retrospective with both learners. It is a **signal, not a gate** — it does not
block P4. Four questions:

1. What did you understand about the domain that you did not know at P0?
2. Where was Muse genuinely useful, and where did it get in the way?
3. Which gate did you pass with difficulty, and why?
4. What would you change about the methodology itself?

Output: `openspec/_methodology/migration-track/runs/<run-id>/mid-run-retro.md`.
This is the methodology's earliest in-run signal of whether it is working —
recorded long before the 30-day delayed test (§8.3) can report.

## 6. Preflight (P0)

P0 exists because several risks the methodology must manage cannot be left to
surface mid-run.

### 6.1 Cost cap and budget telemetry

Set a hard ceiling on Anthropic spend per canvas, hooked into the studio's
existing per-canvas budget infrastructure. When approaching the cap, Muse posts
an alert in the canvas and pauses new work until extended. This is not optional —
without it, long-running multi-session work has no natural brake against runaway
cost.

### 6.2 Tooling check

Verify Muse can read the legacy OpenSpec (if any) and the source repo. Resolve
the path — CLI against an external repo, guided session, or hybrid. Record
findings in the run's Linear epic. Without this, P3 grinds to a halt mid-cluster.

### 6.3 Validation harness — the golden set

Before any learner work begins, Pavel (or a studio architect) assembles a
**golden set**: 10–15 claims about the product, roughly half true and half
false, for which Pavel knows the ground truth. Muse in reviewer mode is asked to
verdict each. Two metrics:

- **Precision on true claims** (Muse says "supported" when it is): ≥ 80%
- **Recall on false claims** (Muse says "contradicted" when it is): ≥ 80%

Below either threshold, the methodology is not ready for the run. This is the
only defense against the tool-call hallucination failure mode — where the
reviewing model confidently returns a verdict it cannot actually support. A Muse
that hallucinates verdicts at the comprehension gate misleads the learner, and
the entire track corrodes silently.

The golden set is committed to the run's folder and rerun at the start of every
phase, not just P0. Drift in Muse's accuracy is a leading indicator of
methodology fatigue.

### 6.4 Scope decision

Pavel and the learners agree explicitly on what is in scope and what is
excluded. Legitimate exclusions: dead admin tools, a deprecated API surface, an
experimental feature that never shipped. Each exclusion is logged with a reason.
This prevents P3–P5 from drowning under everything.

### 6.5 Session discipline rules

Stated up front and enforced by Muse on every session entry:

- Max session length: 90 minutes of active work, or until context exceeds 80K
  tokens, whichever comes first.
- Every session ends with Muse writing a 200–300 word **session snapshot**: what
  was done, what is next, what is open. This snapshot loads at the start of the
  next session instead of the full prior conversation.
- Long phases (P3 clusters, P5 surfaces) explicitly span multiple sessions.
  There is no expectation of finishing a cluster in one sitting.

This is the architectural mitigation for multi-step degradation — the
well-documented decay in model reliability across long multi-step sessions. It
is not negotiable.

## 7. The two gates

A phase's checkpoint is met when both gates hold.

### 7.1 Artifact gate (machine-checkable)

The phase's artifact exists in the repo, is valid, and is committed to the run's
branch. Examples:

- P2 — `domain-modeling.md` committed
- P3.x — every entity in the cluster has a dossier and a rewritten spec under
  `openspec/specs/`
- P4 — relationship spec committed and valid
- P5 — screen spec per identified surface
- P6 — migration plan committed

### 7.2 Comprehension gate (examiner-judged)

This is the gate the methodology lives or dies on. After the artifact gate
passes, Muse runs the comprehension gate as examiner.

**Three questions per learner, one at each level, in order:**

1. **Structural question** — recall and organisation of the phase's content.
   "Name the entities in this cluster and the role of each in one sentence."
   Tests: did you read the dossier and the spec?
2. **Application question** — Muse pulls a fresh code snippet from the product
   (filtered to phase-relevant paths). "This is which entity? Why here and not
   elsewhere?" Tests: can you read the code through the spec you wrote?
3. **Synthesis question** — Muse poses a hypothetical extension. "If the product
   added Shift tracking, which cluster would it belong to? What would change in
   Worker?" Tests: do you understand the boundary, not just the contents?

Questions are pulled from a phase question bank (in
`openspec/_methodology/migration-track/question-banks/`) and rotated per learner
so the two learners never get the same question.

**Verdict per question:** `pass / partial / fail`.

- **Pass** — answer is correct, in the learner's own words, no spec citation.
- **Partial** — answer is technically correct but incomplete or visibly read
  from the spec. Muse names the gap, asks one follow-up; if still partial, the
  question is `partial`.
- **Fail** — answer is materially wrong, or the learner cannot answer at all.

**Phase verdict:**

- 3/3 pass → comprehension PASS.
- 2 pass + 1 partial → **soft fail**: 24h cooldown, new questions, retake.
- ≥ 1 fail → **hard fail**: 48h cooldown, dossier review required, retake.
- 2 consecutive hard fails on the same phase → methodology-level red flag, pause
  the run, retrospective with Pavel.

**Both learners must pass independently** for the phase to close.

The cooldown is not a punishment. It exists so the learner can re-engage with
primary sources (code, legacy spec, dossier) between attempts. Without cooldown,
learners pattern-match to the question form and pass on a third try without
understanding.

### 7.3 Disagreement between learners

When the two learners disagree on a topic ("Worker is a person" vs. "Worker is a
project role"), the disagreement is preserved in the dossier as
`## Open: <position A>` / `## Open: <position B>`. This is **not** a gate
failure — disagreement is data. Resolution happens in P6, where the target
architecture decides.

### 7.4 Recording

Every gate run produces a record at
`openspec/_methodology/migration-track/runs/<run-id>/P<N>-<phase>-gate-record.md`
with: questions asked, answers given, verdict per question, rationale. One file
per learner per retake. The Linear sub-issue receives a summary comment with a
link to the commit.

These records are the methodology's primary feedback corpus — how Migration
Track improves between runs.

## 8. Drift safeguards

The methodology's value collapses if the spec it produces silently rots over the
following months. Three mechanisms protect against this.

### 8.1 Periodic golden set rerun

The same golden set from P0 is rerun at the start of each phase and at the end
of the run. Falling precision/recall is a leading indicator.

### 8.2 Entity-level drift check

Each entity spec is intended to get a paired drift check, delivered by the
planned `add-spec-drift-ci` follow-up: a script that runs weekly and asserts
that occurrences of the entity name in code materially match occurrences in the
spec. Threshold and fuzziness are tuned per spec. Failures generate Linear
issues; they do not auto-merge fixes — a human reviews. The script does not yet
exist; this section describes the intended safeguard, not a shipped tool.

### 8.3 30-day delayed comprehension test

Thirty days after P6 closes, Muse poses one structural and one synthesis
question (no application question, since the code has likely moved) per learner
about a randomly chosen entity. If the learner cannot answer, the run's
comprehension gate failed — even if it passed in real time. This metric drives
methodology revision: if half of all runs fail the 30-day test, the gates are
spec theatre and need redesign.

## 9. Pilot run #1 — vivod

vivod (construction / facade-progress SaaS) is the first run, Linear epic
**VVD-1**. ~30 entities across three surfaces (`platform`, `worker`, `portal`).
P3 splits into six cluster sub-issues:

1. Project frame — Organization, Project, Building, Facade, Block
2. Workforce — Worker, ProjectWorker, Assignment
3. Work tracking — WorkReport, WorkReportItem, MonthlyCount, TaskTemplate, TemplateField, TaskSubmission, FieldResponse
4. Commercial — Client, Contract, ProjectParticipant, Payment, Lead
5. Materials — Stone, MaterialItem, MaterialIssue, MaterialReturn
6. Analysis & documents — FacadeAnalysis, Document

Learners: Pavel and Misha (`michael@vivod.ai`). Surface: the `Vivod AI` canvas
in `app.rapoport.studio`, run with Muse in reviewer mode.

vivod already has a detailed, formal, AI-generated OpenSpec corpus (REQ-numbered
requirements across 16 capability specs). That corpus is a source material for
reconstruction (§1), not a thing to discard — it is read alongside the codebase
and the founder's operational knowledge, and rewritten in studio dialect.

The pilot validates the methodology itself. Every friction point, every gate
that needed remediation, every 30-day test failure feeds back into this
document. v0.1 → v0.2 happens after VVD-1.

## 10. Muse reviewer mode

Reviewer is a distinct Muse mode from Architect, following the same structural
pattern as the existing `architect` and `maestro` modes
(`packages/muse/src/modes/<mode>/` with `runner.ts`, `types.ts`, `tools.ts`,
`prompts/system.ts`). Defined by:

- **System prompt:** a string constant exported from
  `packages/muse/src/modes/reviewer/prompts/system.ts` (embedded for Workers
  deployment, as with the other modes).
- **Tools:** cross-corpus retrieval over code, current spec, and legacy spec; a
  claim-verdict heuristic over that retrieval; Linear comment/read; repo commit
  for gate records.
- **Behaviour contract:** never authors specs; only reviews, challenges,
  examines. Refuses generative requests with a redirect to Architect.
- **Mode entry:** at session start Muse states "Reviewer mode, run VVD-1, phase
  P{N}: {status}." This forces the learner to see the contract.

This is a mode of Muse, not a seventh agent — a behaviour contract, not a
separate model. It is built by the `add-muse-reviewer-mode` follow-up change;
Migration Track cannot run a pilot until it ships.

## 11. Care principles

A transition into a new working environment carries known risks. People migrate
better when the risks are named, not hidden. The five principles below shape how
Muse behaves and how the studio holds the run.

### 11.1 The dependency trap

Around P4 and P5, a learner often crosses into over-reliance: every question
goes to Muse, every decision waits for Muse, original thought stops emerging.
This is the failure mode of any powerful tool. Migration Track is not exempt.

Muse counters this in three ways:

- **Forced pauses.** From P4 onward, Muse refuses to answer at least one
  structural question per session, redirecting to the learner: "You have enough
  to answer this. What do you think?"
- **Solo dossier requirement.** From the third entity cluster in P3 onward, the
  dossier's first draft is written without Muse in the conversation. Muse joins
  only for review. The learner's unaided thinking is the evaluated unit.
- **30-day delayed test (§8.3).** Stated also here because its real purpose is
  detecting unhealthy dependency: a learner whose structural thinking evaporated
  within thirty days never internalised it.

### 11.2 The plausibility trap

Spec language is patternable. After a few weeks, learners can produce text that
*looks* like a spec without holding the structure beneath it. This is the same
risk that doomed waterfall and most documentation-first initiatives: text that
exists, no one reads, no one updates, mistaken for shared understanding.

The comprehension gate's application and synthesis levels (§7.2) are the only
defense. They cannot be passed by pattern-matching to prior answers, because the
source material — a fresh code snippet, a novel hypothetical — varies per
attempt. If the gates are softened for time or pressure, the methodology
degrades to spec theatre.

The studio's responsibility: never soften a gate. A run that finishes faster by
relaxing gates produces a finished corpus and an unchanged founder. That is the
failure we are most likely to miss in real time and most likely to regret in six
months.

### 11.3 The emotional load

Migration Track produces, by design, a moment around P2 and P3 where the learner
confronts how little they understand of what they built. This is uncomfortable.
It is also the most productive moment of the run, because it is when structural
thinking starts to actually form.

Studio responsibility:

- **Name this in advance** (§2.4).
- **Normalise it when it happens.** "This is the work. You are at the threshold.
  Stay with it."
- **Never minimise it with reassurance.** The discomfort is doing the teaching;
  removing it removes the lesson.
- **Hold the learner at the threshold long enough to cross.** Not by pushing
  harder, but by not letting them retreat into the old language.

Muse's tone here is critical. Reviewer mode is calibrated to be respectful of
fear, slow enough for safety, never patronising, and never falsely warm. A
learner who senses performance from Muse stops trusting the environment.

### 11.4 Modular ownership and partnership

A founder who completes Migration Track ends with a product described as
modules — discrete capabilities with named boundaries. This is not only an
architectural outcome. It is a **social** outcome.

When a product is one indivisible thing, it can only be owned alone, defended
alone, scaled alone. When a product is modules with boundaries, each module is
something that a partner can take responsibility for. The founder no longer has
to carry everything. They have to find people they trust for specific parts.

Migration Track encourages this directly. In P6 (target + migration plan), the
founder is asked not only "what does the new architecture look like" but also
"who, other than you, will own this module?" The answer can be "me, for now" —
that is honest. But the question is asked, in every module, every time.

The studio demonstrates this on itself. Rapoport Studio is a network of
specialists, not a single team. Each capability we bring to a founder —
research, architecture, frontend, AI engineering — has a name and a face
attached to it. We are the first partner the founder works with structurally.
We are not intended to be the last.

A founder who finishes Migration Track and still believes they must do
everything themselves has missed the point of the modularisation. The studio's
responsibility includes pointing this out, kindly, in P6 if needed.

### 11.5 The exit

Migration Track must end. Around week ten to twelve, the learner should begin
finding Muse's clarifications redundant, the comprehension gates easy, the
structural questions answerable without help. At that point, P6 includes an
explicit conversation: **what do you no longer need from us?**

A founder who continues depending on studio infrastructure after P6 is not a
customer success — they are a structural failure of the methodology. The track's
mark of completion is the founder operating in the new environment on their own,
with their own partners around the modules of their own product.

The studio's success metric, both for the methodology and for the platform as a
whole, is **the rate at which we become unnecessary to the founders who passed
through us**. We measure this through the 30-day delayed test (§8.3), through
follow-up six months after handoff, and through the founder's own report of how
much of their structural work they did without us in the quarter after exit.

A studio that grows by keeping founders dependent has failed at its actual
purpose. We will know if we are drifting toward that failure mode because the
exit metric will tell us. We commit, in this methodology, to watching it.

## 12. Run lifecycle

```
[New client]
   |
[P0 Preflight]  — cost cap, harness, scope, session rules
   |
[P1 Literacy] → [P2 Domain] → [P3 Entities (per cluster)]
   |
[Mid-run checkpoint P3→P4]  — signal, not gate
   |
[P4 Relationships] → [P5 Surfaces] → [P6 Migration plan]
   |
[Handoff] → [30-day delayed test] → [run retrospective → v0.x]
```

Run state lives in one Linear epic. The epic is the single source of truth for
run progress; there is no parallel state file. Muse reads the epic at every
session entry.

## 13. Open questions

- **Q1** — Question banks: generic (reusable across runs) or run-specific
  (regenerated per product)? Working default: run-specific for
  application/synthesis questions, generic for structural. Revisit after VVD-1.
- **Q2** — Validation harness location: in the run folder, or in the product
  repo? Working default: run folder, with a copy committed to the product repo
  at handoff for ongoing drift monitoring.
- **Q3** — When does P5 (surfaces) honestly start versus when is it speculative?
  Some products have surfaces that are themselves experimental and do not
  deserve spec real estate. The P0 scope decision covers this, but the rubric is
  still rough.
