# Chat surface — screen-reader walkthrough runbook

> Manual artefact complementing the automated axe-core matrix in
> `packages/ui/__tests__/components/chat-composition.axe.test.tsx`.
> Re-validate when the chat state surface changes (new ComposerState,
> new banner kind, new ChatMessage status, new block kind).

---

## Why a runbook, not just axe

`axe-core` catches structural violations — missing `aria-label`, broken
`aria-allowed-attr`, mislabeled landmarks, role contradictions. It does
NOT verify what a screen-reader user *hears* during a turn lifecycle.
For example:

* Does the assistant's `tool-status` block announce when a tool starts?
* Does a network-error banner interrupt the SR cursor mid-paragraph, or
  wait for the user to navigate to it?
* Does the focus jump back to the composer after a partial assistant
  response when the user presses Stop?

These questions need a human + an SR. The runbook records what the user
hears, what they should hear, and any gap between the two.

**Cadence:** re-run quarterly, and whenever a chat state surface
changes (a new ComposerState, a new banner kind, a new ChatMessage
`status`, a new ContentBlock kind). The automated matrix gates CI; this
runbook gates spec amendment.

**Authority:** D-A11Y-4 in `openspec/changes/add-chat-a11y-gate/design.md`
ratifies the runbook lives under `_methodology/research/` (not under
`current/`) so quarterly re-runs do not block the change archive.

---

## Tools

* **NVDA 2025.x** (Windows) — paired with Chrome 140+. Default voice;
  default rate. Capture with NVDA's speech-viewer log + a manual
  transcript of unexpected behaviour.
* **VoiceOver 14** (macOS) — paired with Safari 18+. Verbosity = default;
  Web Rotor as the cross-check tool.
* **axe-core runner** (optional) — `axe.run(container, { reporter:
  "v2" })` to capture the aria-tree dump alongside the SR transcript.

---

## Per-state walkthrough — composer × viewer role

### `idle` × owner

* Sequence: page load → Tab into composer → Tab past composer.
* NVDA expected utterance:
  > "Verification matrix. region. Threads list. New thread button.
  >  Founder. Tighten the proposal. Muse Architect. Acknowledged. The
  >  proposal sharpens around three decisions: scope, surface,
  >  persistence. Composer. Edit. Type a message. Send button."
* VoiceOver expected utterance (similar shape; persona names rendered
  by the assistant chip read aloud as plain text).
* Failure modes to watch for:
  * Composer announces as "edit, type a message" — must NOT contain
    "disabled" (it isn't).
  * Send button announces as a button — must NOT announce as "image"
    (icon-only without aria-label).

### `focused` × owner

* Sequence: page load → Click composer to focus.
* NVDA expected utterance:
  > "Composer. Edit. <empty>. Type a message. Founder. Send. Press
  >  Enter to send."
* The visual focus ring change is non-semantic; SR users perceive only
  the focus shift.

### `receiving-voice` × owner

* Sequence: dictation tool active (Wispr / Dragon / NVDA dictation).
* NVDA expected utterance:
  > "Composer. Edit. Dictating. <transcript so far>."
* Failure modes:
  * "Dictating" suffix on the composer pill must be present (not just
    visual). Per `voice_input_via_telegram.md`, in-browser voice is not
    canonical; this surface is for parity coverage only.
  * Per RAP-770 M6 closure: NO "· WISPR" suffix. SR should announce
    "Dictating" alone.

### `streaming` × owner

* Sequence: user submits → assistant turn streams.
* NVDA expected utterance:
  > "<assistant text streaming in>. Composer disabled. Stop button.
  >  Press Stop to abort."
* Failure modes:
  * Send button replaced by Stop — must announce as a button labelled
    "Stop", not "image" or "icon".
  * Streaming text should NOT re-announce the entire conversation on
    each token (live-region politeness must be `polite`, not
    `assertive`).
  * `aria-busy="true"` on the message list must propagate so NVDA
    indicates "busy" rather than reading every diff.

### `disabled` × owner

* Sequence: page load with composer disabled prop set.
* NVDA expected utterance:
  > "Composer. Edit. Disabled. Type a message."
* Failure modes:
  * Composer must announce "disabled" (the `disabled` attribute, not a
    CSS-only treatment).

### `readonly` × owner

* Sequence: page load with viewer in read-only role.
* NVDA expected utterance:
  > "Composer. Edit. Read only. Type a message."
* The `readonly` attribute differs from `disabled` — SR users must hear
  the distinction (read-only is browsable, disabled is not).

### × collaborator and × client

Same composer behaviour as owner except:

* `client` viewer: memory sidebar is DOM-absent (per `ChatPrimitive`
  contract: "viewerRole: 'client' → sidebar DOM-absent"). NVDA must NOT
  announce a "memory sidebar" landmark.
* Message-actions menu (Copy / Regenerate / Edit / Share / Branch) is
  role-gated. Per-role visibility documented in the chat primitive
  spec; SR users must hear only the actions they can perform.

---

## Per-banner walkthrough

All five banner kinds use the same DOM shape:
`<div role="alert" data-banner-kind="...">` with a human-readable
message. SR utterance announces the banner immediately on appearance
(`role="alert"` is implicit `aria-live="assertive"`).

### `network-error`

* Triggered by: server 5xx or fetch abort.
* NVDA expected utterance (on appearance):
  > "Network error. Retry available."
* Failure modes:
  * Banner must NOT auto-dismiss without user action — SR users may
    not see a transient toast.
  * Retry button must be focusable + reachable via Tab.

### `rate-limit`

* Triggered by: HTTP 429 from Anthropic.
* NVDA expected utterance:
  > "Rate limit reached. Try again in 35 seconds."
* The countdown is decorative; SR users hear the static message. A
  live-region countdown (announcing every second) is anti-pattern and
  must NOT ship.

### `safety-block`

* Triggered by: Anthropic refusal event.
* NVDA expected utterance:
  > "Message refused by safety policy. Rephrase to continue."

### `context-limit`

* Triggered by: token-count exceeded.
* NVDA expected utterance:
  > "Context window full. Trim to continue."

### `reconnecting`

* Triggered by: Realtime bridge disconnect.
* NVDA expected utterance:
  > "Reconnecting."
* Note: this is a transient state. On reconnect, the banner unmounts.
  SR users hear the announcement on mount; the unmount is silent (no
  "reconnect resolved" announcement — keeps the live-region quiet).

---

## Per-state walkthrough — empty thread

When `messages.length === 0`:

* NVDA expected utterance (scanning the message-list region):
  > "Thread list. No messages."
* Suggestion chips (when present): each chip is a button with the chip
  text as the accessible name.
* Failure modes:
  * Empty state must NOT announce as "empty" with no description —
    SR users need a hint about what to do next.

---

## Aria-tree dumps

`axe.run(container, {})` with `reporter: "v2"` returns a serialised
violations list. Pair with the SR transcript for each combo; commit the
dump alongside this runbook the first time a regression surfaces in
review. The dumps are NOT committed by default — they're noisy and
mostly redundant with the automated matrix.

Sample dump shape (for the record):

```jsonc
{
  "violations": [],
  "passes": [
    {"id": "aria-allowed-attr", "nodes": [/* ... */]},
    {"id": "aria-required-attr", "nodes": [/* ... */]},
    {"id": "button-name", "nodes": [/* ... */]}
  ],
  "incomplete": []
}
```

---

## Known gaps

* **Reduced-motion** — out of scope here; covered by the design-token
  motion-mode contract (`design-motion.md § Two motion modes`). The
  axe matrix verifies static-DOM accessibility; live-motion behaviour
  is tested via the design-system test surface separately.
* **P5 error taxonomy (RAP-1094)** — when shipped, the four typed
  error kinds (`rate-limit` / `cost-cap` / `safety` / `offline`) will
  replace the four ReactNode banner kinds in this runbook. The
  per-kind SR utterance shape stays the same; only the prop pathway
  changes.
* **P8 keyboard nav (RAP-1099)** — `j` / `k` traversal across 1000
  messages is verified by an e2e test in P8, not by this runbook.

---

## Change log

* **2026-05-20 — initial commit (RAP-1091 / P4).**
  Matrix shape: 6 composer × 3 viewer + 5 banner × 3 viewer + 1 empty
  × 3 viewer = 36 cells. Re-validated when state surface changes.
