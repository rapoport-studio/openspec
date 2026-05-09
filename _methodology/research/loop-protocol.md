# Loop protocol — 15–20-min loop methodology

> **Status: research-level register, validation-pending.** Originally
> introduced under [RAP-119](https://linear.app/rapoport-studio-slr/issue/RAP-119)
> `add-team-onboarding-canvas` to be validated across Sessions 1–4
> (cancelled 2026-05-08 — engagement deprioritised against MVP v1, see
> [`openspec/archive/2026-05-08-add-team-onboarding-canvas/`](../../archive/2026-05-08-add-team-onboarding-canvas/)).
> Promoted from change-local research to corpus-level register so the
> protocol is reusable for any future canvas (1:1 or multi-author).
> Validation criteria § *Validation criteria* still apply when first
> applied at scale; corpus-promotion to a methodology spec remains an
> open path via a separate future change once empirical use accumulates.

---

## Anatomy of one loop (15–20 min)

A loop is a single unit of work inside a session. Each loop has five
phases. Together they round-trip from input to next-loop seed.

| Phase | Time | What happens |
|---|---|---|
| **a. Input** | ~2 min | A quote, question, or draft is brought in. Source: harvest, prior loop, member-on-the-spot. |
| **b. Muse turn** | ~3 min | Muse responds inside the canvas. Architect persona; course-meta directive active (one-line "what I'm doing" beat). |
| **c. Draft** | ~7–10 min | Owner edits the draft live. Group watches the screen. Other members can suggest in voice. |
| **d. Critique** | ~2–3 min | Group critique. Two passes: factual (is this true?) → framing (does this lead the right way?). Pavel keeps the timer. |
| **e. Next** | ~1 min | Capture: what did this loop produce? What does loop N+1 need as input? Written into harvest as the loop's exit-line. |

**Total:** 15–20 min per loop. Loops back-to-back; no interstitial
chatter beyond next-loop seed.

---

## Session-level skeleton

Each ~90 min session = Loop 0 + 3–4 working loops + Close.

```
00:00 ─ 00:10  Loop 0     framing / read-back / ratification
00:10 ─ 00:30  Loop 1     working
00:30 ─ 00:50  Loop 2     working
00:50 ─ 01:10  Loop 3     working
01:10 ─ 01:25  Loop 4     working (optional, if pace allows)
01:25 ─ 01:30  Close      homework + next-loop seed
```

**Loop 0** is special. It is the bridge from "what we already wrote"
(harvest, last session's exit, homework) to "what we are about to do".
Always read-back, always live-ratification of any pre-existing material.
Never opens with new content.

**Close** is also special. It is the bridge to the next session. Two
artifacts produced: homework prompt (1 line per member) + seed for next
session's Loop 1.

---

## When to deviate

- **Short loop (~10 min)** — for quick clarifications, naming
  decisions, or yes/no calls. Owner says "this should be a short loop"
  before it starts. Skip phase (c) — go input → Muse → critique → next.
- **Long loop (~25 min)** — for first-draft writing of a substantive
  paragraph or table. Owner says "I need a long loop" before starting.
  Phase (c) expands; phase (d) cuts to 1 min.
- **Parallel loops** — for hands-on sessions where each member works in
  their own canvas, facilitator circulates. Phase (d) critique still
  happens, but only at the end of the parallel block, not after each
  loop.

Owner's call. The default — 15–20 min, sequential — wins arguments.

---

## Why 15–20

- Long enough to produce a real draft (not just a sentence).
- Short enough that everyone stays attentive (matches typical
  attention span before drift).
- Short enough that 4 substantive loops fit in 90 min after open/close.
- Short enough that a bad loop is recoverable by the next loop.

Empirical: this matches the rhythm Pavel already uses in 1:1 canvases
(prior conversations confirm it). The team-of-N case (multi-author) is
the new variable. First multi-author engagement validates the rhythm
at scale.

---

## What this protocol is NOT

- It is not a script. The five-phase anatomy is a frame; the *content*
  of each loop is decided in the moment.
- It is not a Pomodoro. Loops are work units, not breaks.
- It is not a Scrum / Kanban / Agile ceremony. It's a single facilitator
  pattern for collaborative writing.
- It is not corpus-level methodology yet. It is a research-level
  register. Promotion to a methodology spec (its own change with
  `openspec/current/<capability>.md` amendment) is a separate, later
  proposal once empirical use accumulates.

---

## Validation criteria

When first applied at scale (multi-author session series), ask:

1. Did the loop-skeleton hold across all sessions?
2. Did members find the rhythm productive — or felt forced?
3. Did Loop 0 ratification feel like the right opening — or felt
   ceremonial?
4. Were the close-loop seeds actually used by the next session?
5. Could a different facilitator (not Pavel) run a session using this
   spec alone?

If 4-of-5 yes → propose `add-loop-protocol-methodology` change.
If less → keep as research. Document why.

---

## Provenance

- Originated as `openspec/archive/2026-05-08-add-team-onboarding-canvas/research/loop-protocol.md` on 2026-05-07 (PR #262); archived 2026-05-08.
- Referenced from D10 of the same change's `design.md`: session structure is loop-based, not minute-by-minute.
- Engagement cancelled 2026-05-08 without Sessions 1–4 running. Protocol promoted to corpus-level register so it survives outside the engagement frame.
