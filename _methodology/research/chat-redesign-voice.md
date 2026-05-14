# Chat redesign — voice-first input & streaming UX (source stream 3 of 4)

> 2026-05-14 · feeds [synthesis](chat-redesign-synthesis.md) · sister streams: [patterns](chat-redesign-patterns.md), [a11y](chat-redesign-a11y.md), [adjacent](chat-redesign-adjacent.md).
>
> Coverage: Wispr Flow deep-dive, voice-first composer design, streaming-response UX patterns. 2025–2026 state.

---

## 1. Wispr Flow — deep dive

**What it is.** A system-wide dictation overlay (not a browser extension, not a web API) for Mac, Windows, iOS, and Android. The Android build is the only one with public technical docs on how text injection works — explicitly: *"Android does not provide a standard API for third-party apps to detect text field focus or insert text into other apps. To work around this limitation, Wispr Flow uses the Android Accessibility Service…"* Inserts text via "accessibility input connection" without replacing the keyboard.

**Mac/Windows behavior (inferred + reported).** Consensus across reviews and bug reports: Flow uses **OS accessibility APIs + simulated paste**, not raw keystroke synthesis. Evidence: it breaks on **non-QWERTY layouts** (Dvorak/Colemak/non-Latin) — classic signature of `CGEventKeyboardSetUnicodeString`-style injection mixed with clipboard paste. Zed editor issue `zed-industries/zed#40647` reports Flow cannot type into Zed's main editor or AI panel but works in the command palette, consistent with an a11y-tree text-field discovery that fails on custom-painted text surfaces. Works in both `<textarea>` and `contenteditable` based on user reports across Notion, Linear, Slack, ChatGPT and Claude.ai — but breaks on apps that re-render input on every keystroke or intercept paste.

**Pricing (2026).** $15/mo or $144/yr (Pro). Free tier = 2,000 words/week. Cloud-only — audio goes to Wispr's servers.

**Known web-app quirks.**

- Electron-based, RAM-heavy.
- Reliability degradation post-trial went viral on r/ProductivityApps in early 2026; CTO publicly apologized.
- Screen-capture context — Flow screenshots the active window to "understand" context. Power users in dev/proprietary contexts treat this as a non-starter.
- Auto-startup — added itself to login items aggressively until 2026 update.

**What power users say makes an input "good for Wispr":**

- Use a plain `<textarea>` or a plain `contenteditable="true"` div. Both work. Don't roll your own IME logic.
- **Do not listen to `keydown` to drive state.** Listen to `input` and `change`. Wispr injects without firing per-character `keydown`, so `keydown`-driven autocomplete/formatters silently drop dictated text.
- **Do not intercept `paste`.** Wispr's primary desktop insertion path appears to be synthetic paste; pages that `stopPropagation()` on paste lose text.
- Avoid cross-origin iframes for the composer.
- Avoid React state that re-mounts the input on each keystroke — collapses cursor, Wispr's next-token insert lands wrong.

**SDK / dictation-active detection.** **None public.** `wisprflow.ai/developers` mentions an unspecified "Dictation API" in marketing copy but exposes no docs, no JS hook, no postMessage event. The web app is fully opaque to whether Wispr is currently dictating. Only signal: standard browser `input` event firing without a preceding `keydown`.

**Hotkeys (default).** Mac push-to-talk = hold `Fn` (or `Ctrl+Opt` on external keyboards); hands-free toggle = `Fn+Space`; Command Mode = `Fn+Ctrl`; cancel = `Esc`. Windows PTT = `Ctrl+Win`.

**Roadmap 2026.** $30M Series A led by Menlo Ventures; expanding desktop+mobile, deeper integrations, "enterprise-ready compliance," and MCP support. No public dev-platform commitment.

## 2. Competing voice-first field

- **Superwhisper** — system-wide, **on-device Whisper**, zero cloud, $8.49/mo or $84.99/yr. Privacy-first pick.
- **MacWhisper** — **file-based** transcription, not real-time dictation. Different category.
- **ElevenLabs Scribe v2 Realtime** — 150ms latency speech-to-text API, 90+ languages. Leader for *embedded* dictation.
- **Apple Intelligence Dictation** — on-device, Neural Engine, with Writing Tools as separate post-dictation layer.
- **OpenAI Realtime API + Hume EVI** — speech-to-speech, not dictation. EVI optimizes emotional fidelity; Realtime optimizes general reasoning.
- **Web Speech API** — Chromium-only in practice (Safari partial 14.1+, Firefox flag-gated). Not deprecated, not viable as primary.
- **Granola / Otter / Krisp** — meeting-transcription, not composer-injection. Not direct competition.

Winners: Wispr (polish + mindshare), Superwhisper (privacy), ElevenLabs Scribe (API). Stalled: Web Speech API.

## 3. Composer design for dictation

- **`<textarea>` is the safest substrate.** `contenteditable` works but every framework adds quirks (Slate/Lexical/ProseMirror each have known paste-handling edge cases that have shipped Wispr bugs).
- **Auto-punctuation:** Wispr inserts already-punctuated text. Don't run a second auto-punctuation pass — collisions produce double periods.
- **"New paragraph" voice command:** Wispr emits a real `\n\n`. Honor it. Don't collapse whitespace on input.
- **Undo/redo landmines:** Wispr inserts in one or many `input` events. Bundled-undo-history libs may collapse the whole dictation into a single undo step — that's the right behavior. Test by dictating a paragraph and hitting Cmd+Z once; if everything vanishes, good.
- **Filler stripping:** Wispr removes "um/uh" server-side. Never show "raw" then "cleaned" — only cleaned text arrives.
- **Lag-behind-speech:** Wispr's bubble shows its own indicator. The composer should show **no text + a subtle pending state** during the gap.

## 4. Multimodal input (voice + paste + drag-drop)

- During an active dictation hold, pasting an image should **queue** rather than interrupt.
- Focus should never auto-move during dictation; if the user drops a file, attach it as a pending media chip below the composer, keep cursor in the textarea.
- Mode-switching cue: a single quiet indicator strip below the input.

## 5. "Dictation in progress" affordances

- **Don't duplicate the system indicator.** Mac shows its own mic glyph; Wispr shows its Flow Pill. A third in-app indicator is noise.
- The single legitimate in-composer signal: a **subtle border-color or focus-ring change** while the textarea is receiving non-keyboard `input` events. Detectable heuristically: `input` event without preceding `keydown` ≈ dictation. This is the only opaque-Wispr signal available.

## 6. Streaming response UX (2026 norms)

- **Render granularity:** word-by-word is the sweet spot. Token-by-token reveals BPE artifacts ("th", "ank", " you"). Block-replace feels janky. Buffer until whitespace, then flush.
- **Markdown:** buffer until closing fence/bracket before rendering code blocks; partial render for prose is fine.
- **Cursor animation:** a thin blinking vertical bar (Claude.ai style) or a solid square (ChatGPT style). "None" is wrong in 2026 — users use it as the streaming-vs-done signal.
- **Abort:** `Esc` to stop is the emerging norm; ChatGPT, Claude, Cursor all converged on a visible Stop button + Esc.
- **Keep-typing-while-streaming:** 2026 consensus has shifted to **yes, allow it** — composer stays interactive, next message is queued.
- **Snappy latency targets:** TTFT under ~600ms feels instant (Claude Haiku 4.5 = 597ms TTFT). Under 800ms = "fast." Over 2s = "broken." Throughput: 100+ tokens/s is the floor for "smooth"; Gemini 2.5 Flash sets the bar at 146–173 t/s.

## 7. Hands-free trade-offs

- **Hold-fn-to-talk** (Wispr default) is the lowest-error model — no wake word, no false start, ends precisely on release.
- **Toggle/hands-free** is for long-form drafting but requires explicit "stop."
- **Always-listening** has lost the 2026 design debate for productivity tools — privacy + battery + accidental capture.
- **Explicit-start button in-app** is dead weight when Wispr/Superwhisper run system-wide. Don't add one; you'll just trigger Wispr on top of itself.

## 8. Multimodal output (voice-out)

Text-out remains the default. Voice-out is becoming a **secondary surface** in 2026 (ElevenLabs v3, OpenAI Realtime, Hume EVI). Leave room in the message data model for an optional `audio_url` per assistant message and a per-message TTS-replay affordance, but don't make voice-out primary unless the product is conversational. Winning pattern: voice-out as a play button on each assistant reply, not as continuous duplex.

## Confidence flags

- Mac/Windows Wispr insertion internals inferred from bug reports + Android docs — Wispr publishes no desktop technical reference.
- No public dictation-active hook exists.
- "Power-user composer rules" synthesized from Dictanote/Voice In guides plus Zed/Wispr issue threads, not Wispr's own docs.

## Sources

- [Wispr Flow — homepage](https://wisprflow.ai/)
- [Wispr Flow — Accessibility Permission on Android](https://docs.wisprflow.ai/articles/7669452251-accessibility-permission-on-android)
- [Wispr Flow — Hotkey Shortcuts docs](https://docs.wisprflow.ai/articles/2612050838-supported-unsupported-keyboard-hotkey-shortcuts)
- [Wispr Flow — Non-QWERTY layout issues](https://docs.wisprflow.ai/articles/1621472516-flow-fails-to-detect-text-fields-or-inserts-incorrectly-on-non-qwerty-keyboard-layouts)
- [Wispr Flow — Developers landing page](https://wisprflow.ai/developers)
- [Wispr Flow — Context Awareness](https://docs.wisprflow.ai/articles/4678293671-feature-context-awareness)
- [Wispr Flow — Pricing](https://wisprflow.ai/pricing)
- [Wispr Flow — Roadmap / Changelog](https://roadmap.wisprflow.ai/changelog)
- [Zed issue #40647 — Wispr Flow text injection bug](https://github.com/zed-industries/zed/issues/40647)
- [eesel — deep-dive Wispr Flow review](https://www.eesel.ai/blog/wispr-flow-review)
- [Ryan Shrott — Why I Cancelled My Wispr Flow Subscription](https://medium.com/@ryanshrott/why-i-cancelled-my-wispr-flow-subscription-and-what-im-using-instead-d783433f4411)
- [Dictanote / Voice In — Make your web app Voice In compatible](https://help.dictanote.co/help/general/make-your-web-app-voice-in-compatible)
- [WhisperClip — comparison vs Superwhisper vs Wispr vs MacWhisper](https://whisperclip.com/blog/posts/whisperclip-vs-superwhisper-wisprflow-macwhisper-which-ai-dictation-fits-your-workflow)
- [Can I Use — Speech Recognition API](https://caniuse.com/speech-recognition)
- [ElevenLabs — Scribe v2 Realtime](https://elevenlabs.io/realtime-speech-to-text)
- [Sara Zan — Can you really interrupt an LLM?](https://www.zansara.dev/posts/2025-06-02-can-you-really-interrupt-an-llm/)
- [thefrontkit — AI Chat UI Best Practices (2026)](https://thefrontkit.com/blogs/ai-chat-ui-best-practices)
- [Apple — Apple Intelligence](https://www.apple.com/apple-intelligence/)
