# Chat redesign — research synthesis

> 2026-05-14 · feeds into RAP-XXX (minimal reusable chat primitive in `@rapoport-studio/ui`).
>
> Source streams: [patterns](chat-redesign-patterns.md) · [a11y](chat-redesign-a11y.md) · [voice](chat-redesign-voice.md) · [adjacent](chat-redesign-adjacent.md).

---

## TL;DR

Loud community critique of chat-as-UI (Pike, Wattenberger, Litt — late 2025) is correct: chat is the wrong **default** for knowledge work. But chat is still right as a **fallback** for ambiguous/exploratory tasks (Lee). Our position: build a **minimal chat primitive**, designed to sit *alongside* non-chat surfaces (Canvas, brand window, project surface), not in place of them. Composer is Wispr-perfect (no SDK exists — design as a perfect target). Accessibility is full WCAG 2.2 AA with a specific `aria-live` discipline for streaming (don't stream into a live region; announce completion only into a pre-primed off-screen region).

---

## I. Big-picture stance

The loudest meta-thread in 2025–2026 chat-UX discourse: **chat is the wrong primary UI for knowledge work.** Allen Pike's *Post-Chat UI* is the canonical statement; Wattenberger's *Boo chatbots* and Litt's *Enough AI Copilots! We need AI HUDs* are the supporting cast. The faults named: no affordances, buried context, isolated responses, no comparison view, the affordance-free textbox as the original sin.

These arguments are **right**, and we build *with* them rather than against:

- Chat is one surface, not the default.
- Non-chat surfaces (Canvas, brand window, project surface) sit alongside, not under chat.
- Inside chat: T3 Chat minimalism, not ChatGPT maximalism.

## II. Decisions the research closed

### Layout

- **Single-column messages + collapsible left rail.** Header almost empty (title + kebab). No model picker in header — ChatGPT's pre-2026 6-item dropdown is the canonical bad example.
- Composer pinned bottom. Footer chrome muted but visible (B2B context — token/cost matters).

### Composer — Wispr-perfect

Wispr Flow has **no public SDK**. On Android (the only documented platform) it injects via the Accessibility Service; on desktop it uses OS accessibility APIs + synthetic paste. The "integration" question is therefore moot — what matters is making the composer the perfect target.

Non-negotiable contract:

- Plain `<textarea>` (or flat `contenteditable`; no custom IME logic).
- Auto-grow, ~40% viewport, then internal scroll.
- Listen to `input`, not `keydown`, for state changes.
- Don't intercept `paste` — Wispr's desktop insertion path uses synthetic paste.
- `Enter` sends, `Shift+Enter` newline (desktop); inverted on mobile.
- Send button **morphs to Stop** during streaming (same slot, different icon/color).
- Model/agent picker inline at composer top, never in header.
- No duplicate "Listening" indicator — Wispr/macOS show their own; we maximally render a subtle focus-ring tint when input fires without preceding keydown.

### Per-message actions

- **Always visible:** `regenerate`, `copy` (ChatGPT's May 2026 hover-only-retry regression is the cautionary tale).
- **Hover-revealed:** `edit` (user msg only), `share-link`, `branch`.
- Regenerate has a dropdown for "regenerate with different model".

### Streaming UX (2026 norm)

- **Word-level granularity.** Buffer to whitespace boundaries before flush; never token-level (visible BPE artifacts).
- **Block-deferred** rendering for code fences, tables (FlowToken / llm-ui pattern).
- **Soft caret at tail** — thin vertical bar in accent color. Static under reduced motion.
- **Composer stays interactive during streaming** — user queues next message. 2026 consensus.
- **Esc + visible Stop button** both abort (converged norm across ChatGPT, Claude, Cursor).
- Latency targets: TTFT <600ms = instant; <800ms = fast; >2s = "broken." Throughput floor 100 t/s.

### Edit-reroll branching

- Default: linear-with-edit-reroll (Claude.ai). Siblings reachable via ← / → chevrons. Users barely notice it's a tree.
- Tree view never as primary UI (consistently fails usability above ~3 splits).

### Slash / Cmd-K split

- `/` → insert-into-this-message (inline directive chips).
- `Cmd-K` → navigate / global actions (jump thread, switch model, switch persona).
- Hard rule: never put model-picker behind `/`. That's `Cmd-K`'s job.

### Multi-author rendering

- Non-human avatar shape (square, persona-tinted) for AI; round for humans.
- Distinct color accent per persona.
- AI uses reactions (✅, 👀) for status; messages only for substance.

### Memory / context

- **Visible sidebar pane** (Claude Projects pattern), workspace-scoped.
- Editable as plain text (Cursor `.cursorrules` style).
- Per-response "what shaped this" pill (ChatGPT Sources pattern, May 2025).
- No invisible carryover between workspaces (Willison's memory-dossier critique).

### Empty / loading / error

- Empty: 3–4 **studio-specific** suggestion chips, never generic placeholders.
- Loading: skeleton lines under assistant avatar, no spinners in bubbles.
- Errors differentiated: safety / rate-limit / network / context-limit / STT failure.
- Always preserve unsent prompt — never make user re-type after error.

## III. Accessibility — WCAG 2.2 AA contract

### Streaming announcement strategy

`role="log"` says the spec but Adrian Roselli's Jan 2026 matrix shows it's flaky across NVDA/JAWS/VoiceOver. **Real pattern:**

1. `<section aria-label="Conversation">` for landmark.
2. Streaming content goes into a **non-live container**.
3. On stream-end, push final summary ("Response complete, N words") into a **separate, pre-primed, off-screen** `aria-live="polite"` region.
4. `aria-busy="true"` on the chat region during streaming; flips to `false` after.
5. Hidden `<h2>` per turn for SR jump-by-heading.
6. **ARIA Notify API** (Edge 136+, polyfill available) as progressive enhancement — replaces live regions for this use case.

### Navigation + focus

- **Roving tabindex** on message list (APG canonical); `↑/↓` between messages, `j/k` as enhancement not replacement.
- Esc cancels stream **and** returns focus to composer (no keyboard trap).

### Reduced motion

`prefers-reduced-motion` + our `appearance.reducedMotion` user setting collapses: caret blink → static, word-fade-in → instant, message-in animation → instant, smooth scroll → instant, skeleton pulse → static.

### Color contrast pitfall

Modern chats often use `#f7f7f8` bubble tint on `#ffffff` — fails 1.4.11 (non-text contrast). Two options: 3:1 border, or rely on avatar + accent as structural boundary. **We chose the latter** (no bubbles, avatar + 2px persona-tinted left rule).

### WCAG SC list relevant to chat

`1.3.5` (Input Purpose), `1.4.3` (Contrast Min), `1.4.11` (Non-text Contrast), `2.1.2` (No Keyboard Trap), `2.4.7` (Focus Visible), `2.4.11` (Focus Not Obscured), `2.5.7` (Dragging Movements), `2.5.8` (Target Size), `3.3.7` (Redundant Entry), `3.3.8` (Accessible Authentication), `4.1.3` (Status Messages).

### APCA

Use as design-time quality gate (Stark / Figma plugin), not regulatory. Catches low-tint background bugs WCAG 2.2 misses. WCAG 3 is still 3–5 years out per Eric Eggert + Adrian Roselli — don't wait.

## IV. Anti-patterns (do not ship)

1. Hover-only critical actions (ChatGPT May 2026 retry regression).
2. Buried model picker.
3. Generic "Something went wrong" — must differentiate.
4. Force-everything-through-chat.
5. Recursive threading (flat-by-design wins per Atwood; even threaded forums disable trees by default).
6. Adaptive UI that changes between sessions (Pike's IntelliMenus analog).
7. Over-disclaimering — industry is *removing* boilerplate safety preambles (medical disclaimers dropped 26.3% → 0.97%).
8. Listening indicator stacked on top of Wispr's.
9. Generic placeholder in empty state ("infinite suggestion chips" anti-pattern).
10. Streaming into `role="log"` without side-channel announce.

## V. Adjacent-field borrowings

| From | What to borrow |
|---|---|
| **T3 Chat** | Empty chrome, keyboard-first, inline model picker, minimalism-as-policy |
| **Claude.ai** | Edit-reroll branching, soft fade-in streaming, project sidebar |
| **Linear** | Agent-as-labeled-participant; chat-inside-the-thing-it's-about for future public surface |
| **Warp** | Block-addressable turns — each message is a navigable, copyable, shareable unit |
| **Superhuman** | `Cmd-K` as universal escape + single-letter shortcuts grammar |
| **Telegram** | Quote-reply (highlight-to-quote part, not whole-message reply) |
| **Perplexity** | Inline `[1]` citations + Sources panel — when retrieval lands |
| **Slack** | Multi-author rules: non-human avatar shape, agent reactions for status |

## VI. Explicit deferrals (v1 does not solve)

- Brand-card content block — leave a slot in the ContentBlock discriminated union; UI later.
- Voice-out / per-message TTS replay — leave `audio_url?` on message model; no UI.
- Multi-pane artifact canvas — Canvas surface owns this separately.
- Branch-as-parallel-tabs — linear-with-checkpoints only in v1.
- Citations panel — added when retrieval ships.
- Multi-agent supervision UI.
- Ambient-agent inbox.
- Public-facing / client surface — single internal surface in v1.

## VII. Trends to watch (2026)

- **Multi-agent panels with supervision UI** (Devin "managed Devins," Replit Agent 4, Cursor tabs) — supervision/verification surface is the unsolved UX problem; "2025 was the year of agents, 2026 is the year of harnesses."
- **Canvas/side-by-side as default**, not opt-in (Claude Artifacts, ChatGPT Canvas, v0).
- **Ambient agents** (Lindy, GitHub Agents, Microsoft "change-driven architecture") — request/response giving way to monitor/react.
- **AI HUDs over Copilots** (Litt) — squigglies, inline suggestions, no dialog box. The strongest minority position in 2026 design discourse.

## VIII. Operating definition of "minimal" in B2B 2026

Single composer · keyboard-first grammar · visible scoped memory · inline citations · one optional side panel for artifacts · AI participants labeled but not isolated · no recursive threading · branching as linear-with-checkpoints. Strip everything else.

---

## Sources

Aggregated across the four source streams (see linked notes). Key essays for arguing the "minimal" stance to stakeholders:

- Allen Pike — [Post-Chat UI](https://allenpike.com/2025/post-chat-llm-ui/)
- Amelia Wattenberger — [Why Chatbots Are Not the Future](https://wattenberger.com/thoughts/boo-chatbots/)
- Geoffrey Litt — [Enough AI Copilots! We need AI HUDs](https://www.geoffreylitt.com/2025/07/27/enough-ai-copilots-we-need-ai-huds)
- Simon Willison — [I really don't like ChatGPT's new memory dossier](https://simonwillison.net/2025/May/21/chatgpt-new-memory/)
- Adrian Roselli — [Live Region Support (Jan 2026)](https://adrianroselli.com/2026/01/live-region-support.html)
- Latent Space — [Building AI×UX with Linus Lee](https://www.latent.space/p/ai-interfaces-and-notion)
