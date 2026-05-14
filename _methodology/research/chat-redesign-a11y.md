# Chat redesign — accessibility standards (source stream 2 of 4)

> 2026-05-14 · feeds [synthesis](chat-redesign-synthesis.md) · sister streams: [patterns](chat-redesign-patterns.md), [voice](chat-redesign-voice.md), [adjacent](chat-redesign-adjacent.md).
>
> Coverage: WCAG 2.2 AA + WCAG 3 (Silver) lookahead for AI chat. Specific ARIA attributes, exact success criteria, named patterns.

---

## 1. ARIA live regions for streaming responses

- **`role="log"`** has implicit `aria-live="polite"` + `aria-atomic="false"`. The W3C ARIA-23 technique explicitly cites chat as the canonical use case. **But** Adrian Roselli's Jan 2026 support matrix shows `log` is the least reliably announced role — JAWS treats it as polite, NVDA inconsistent, VoiceOver buggy on macOS, Narrator/Orca don't handle it specially. Consensus is **contested**: spec says use `log`, real-world tests say it under-delivers.
- **`role="status"`** (implicit `polite` + `aria-atomic="true"`) is more reliably announced, but `atomic=true` will re-read the entire buffer on every token — fatal for a 2000-token stream.
- **`role="alert"`** = implicit `assertive` + `atomic=true`. **Do not use for streamed assistant content** — interrupts user navigation. Reserve for STT errors / safety blocks.
- **Modern verdict for streaming LLMs:** announce only on completion. Pattern from Claude.ai screen-reader analysis and a published Tampermonkey fix: keep streamed text in a non-live container; on stream-end, push the final response (or a "Response complete. N words." summary) into a separate, **pre-primed empty** `aria-live="polite"` region. TetraLogical's "prime first, then update" rule is mandatory — pre-populated live regions don't announce.
- **`aria-relevant`**: leave at default (`additions text`). `removals` is poorly supported.
- **`aria-busy="true"`** during streaming, flip to `false` when done — supported reasonably well and signals "batch in progress."
- **Forward-looking:** the **ARIA Notify API** (`element.ariaNotify(message, {priority})`) shipped behind an Origin Trial in Edge 136 (May 2025) and is explicitly designed to replace live regions for this use case. GitHub publishes a polyfill that falls back to live regions. Worth wiring as progressive enhancement.
- **What ChatGPT / Claude.ai actually do:** both stream into live regions and both ship the "NVDA/JAWS reads partial text repeatedly" bug.

## 2. Real-world screen-reader UX of ChatGPT & Claude.ai

- **Claude.ai (NVDA + Chrome on Windows) documented issues:** no completion announcement; Keyboard Shortcuts dialog doesn't move focus or announce itself; "More Options" popup unannounced with no exit hint; sidebar lacks landmarks; conversation turns lack heading structure (no `h2`/`h3` per turn), so jumping by heading fails.
- **ChatGPT documented issues:** message bar inaccessible to standard SR on first tab; chat history hard to reach; no focus shift to new assistant content on completion. iOS ChatGPT has a known bug where VoiceOver double-tap fails to activate Send.
- **Recommended fixes:** inject hidden `h2`/`h3` headings on each user/assistant turn for SR navigation; dedicated live region for "response complete"; suppress streaming live region.

## 3. Keyboard navigation — modern consensus

- Focus order: composer first when entering the chat view, then a "skip to messages" link, then messages, then header/sidebar.
- **Slack model:** focus the empty composer, press `↑` to enter message navigation, `Page Up`/`Page Down` to jump, `Tab` inside a focused message to reach links/attachments. Most-praised pattern.
- **Discord model:** roving focus ring (Keyboard Mode), arrow keys inside list-like regions, `F6` to jump landmarks.
- **Escape from streaming:** `Esc` should stop generation AND return focus to composer. Don't trap focus during stream.
- **Modal/menu focus traps:** required by APG Dialog pattern; `Esc` must close and restore focus.

## 4. Roving tabindex vs linear tab — verdict

**Roving tabindex wins for the message list.** APG's Keyboard Interface guide is explicit: composite widgets with many items should expose one Tab stop (`tabindex="0"` on the active item, `-1` on the rest) and use arrow keys for intra-list navigation. Linear tab through 200 messages is unusable. Use `↑`/`↓` (or `j`/`k` as enhancement, not replacement — single-letter keys collide with SR quick-nav).

## 5. `prefers-reduced-motion`

Disable: streaming caret blink, message-in slide/fade, smooth scroll-to-bottom (use `scrollIntoView({behavior: 'auto'})`), typing-dot pulse, shimmer/skeleton pulse.

```css
@media (prefers-reduced-motion: reduce) {
  .stream-caret, .typing-dots { animation: none; }
  .msg-enter { transition: none; }
  :root { scroll-behavior: auto; }
}
```

Also mirror in JS: `window.matchMedia('(prefers-reduced-motion: reduce)').matches` to skip JS-driven animations.

## 6. Color contrast

- **WCAG 2.2 still mandatory:** `1.4.3` text 4.5:1 (3:1 for ≥18pt or ≥14pt bold); `1.4.11` non-text 3:1 for UI components, icons, message-bubble edge, focus indicators.
- **Pitfall in modern chats:** assistant-bubble tints `#f7f7f8` on `#ffffff` — fails `1.4.11` (the bubble outline is the only message-boundary signal). Either add 3:1 border, raise the tint, or rely on leading avatar/label.
- **Persona accent dots / status pips:** must hit 3:1 vs adjacent bg.
- **Link color inside messages:** 4.5:1 vs bubble bg AND 3:1 vs surrounding text (1.4.1 doesn't allow color as sole indicator — underline or weight required).
- **APCA / WCAG 3:** APCA is the leading candidate for WCAG 3's "Visual Contrast of Text" but **WCAG 3 is not ready** — Eric Eggert's analysis and Roselli's commentary put release 3–5 years out. axe-core has an open issue (#3325) for APCA support but it's not default. **Recommendation:** ship to WCAG 2.2, additionally run APCA in design tools (Stark, Figma plugins) as a quality gate.

## 7. Voice-input accessibility

- **Listening state:** `aria-pressed="true"` on the mic toggle; announce "Listening" via a primed `role="status"` region on start, "Stopped listening" on end.
- **Live captions during dictation:** render interim STT results in a visible region with `aria-live="polite"` and `aria-atomic="false"` — but throttle to ~500ms or update only on word boundaries to avoid SR flooding.
- **Final transcript:** must be visible and editable before send.
- **STT failure:** `role="alert"` for "Microphone permission denied" / "Couldn't hear that" — rare legitimate uses of `assertive`.
- No formal W3C pattern exists. Web Speech API spec is silent on a11y.

## 8. WCAG 2.2 SC most relevant to chat

- **2.4.11 Focus Not Obscured (Min, AA)** — sticky composer at the bottom must not hide focused messages above. Use `scroll-padding-bottom: <composer-height>` on the scroll container.
- **2.5.8 Target Size (Min, AA)** — 24×24 CSS px for send, mic, message-action icons.
- **2.5.7 Dragging Movements (AA)** — any drag-to-reorder must have a tap/click alternative.
- **3.3.7 Redundant Entry (A)** — don't ask user to re-paste context already supplied.
- **3.3.8 Accessible Authentication (Min, AA)** — magic-link / passkey flow into the chat must not require remembering a code.
- **1.3.5 Identify Input Purpose** — composer textarea: `autocomplete` where applicable; mic button labeled in code, not just by icon.
- **2.1.2 No Keyboard Trap** — streaming must not trap focus; Esc cancels.
- **4.1.3 Status Messages** — formal hook for the "response complete" announcement.
- **2.4.7 Focus Visible** + **2.4.13 Focus Appearance (AAA, optional)** — visible ring on every interactive element including message-row focus when using roving tabindex.

## 9. `role="log"` vs alternatives

Spec says `log`. Practice says it's flaky. **TetraLogical / Sara Soueidan implicit consensus:** if your chat is the page's primary content, use a labeled `<section aria-label="Conversation">` (i.e., `region`) for landmark navigation, and put the *announcement* into a separate, off-screen, pre-primed `aria-live="polite"` region that you write to only on completion. Don't stream into `role="log"` and hope. **Settled** that you need a landmark; **contested** whether `log` adds value over `region` + side-channel announcer.

## 10. Specific audits / first-person reports

- Claude.ai NVDA analysis — dev.to/wiscer "Screen Reader Experience Analysis on claude.ai"
- ChatGPT screen-reader walkthrough — CaptechConsulting "AI & Accessibility: A Look into ChatGPT"
- ChatGPT iOS VoiceOver send-button bug — OpenAI community thread
- ChatGPT a11y survey — Accessible.org "Can ChatGPT Audit Your Website"
- ChatGPT academic case study — arXiv 2501.03572

No published Deque/TPGi audit of ChatGPT/Claude/Gemini/Copilot specifically found. Community-Access "accessibility-agents" repo and JacquelineDMcGraw/claude-a11y exist as practitioner workarounds, not formal audits.

## Sources

- [W3C ARIA APG — Keyboard Interface](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)
- [W3C ARIA-23 — role=log for sequential updates](https://www.w3.org/WAI/WCAG21/Techniques/aria/ARIA23)
- [W3C — Understanding 2.4.11 Focus Not Obscured](https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-minimum.html)
- [Adrian Roselli — Live Region Support (Jan 2026)](https://adrianroselli.com/2026/01/live-region-support.html)
- [TetraLogical — Why are my live regions not working?](https://tetralogical.com/blog/2024/05/01/why-are-my-live-regions-not-working/)
- [Sara Soueidan — Accessible notifications with ARIA Live Regions (Pt 1)](https://www.sarasoueidan.com/blog/accessible-notifications-with-aria-live-regions-part-1/)
- [Sara Soueidan — Pt 2](https://www.sarasoueidan.com/blog/accessible-notifications-with-aria-live-regions-part-2/)
- [TPGi — Screen reader support for ARIA live regions](https://www.tpgi.com/screen-reader-support-aria-live-regions/)
- [MDN — Element.ariaNotify()](https://developer.mozilla.org/en-US/docs/Web/API/Element/ariaNotify)
- [Microsoft Edge Blog — Aria Notify Origin Trial](https://blogs.windows.com/msedgedev/2025/05/05/creating-a-more-accessible-web-with-aria-notify/)
- [dev.to/wiscer — Screen Reader Experience Analysis on claude.ai](https://dev.to/wiscer/screen-reader-experience-analysis-on-claudeai-433a)
- [GitHub — JacquelineDMcGraw/claude-a11y](https://github.com/JacquelineDMcGraw/claude-a11y)
- [CaptechConsulting — AI & Accessibility: ChatGPT](https://www.captechconsulting.com/technical/artificial-intelligence-accessibility-a-look-into-chatgpt)
- [Slack — Use Slack with a screen reader / keyboard nav](https://slack.com/help/articles/360000411963-Use-Slack-with-a-screen-reader)
- [Discord Engineering — App-Wide Keyboard Navigation](https://discord.com/blog/how-discord-implemented-app-wide-keyboard-navigation)
- [Adobe Spectrum — Roving Tab Index](https://opensource.adobe.com/spectrum-web-components/tools/roving-tab-index/)
- [Eric Eggert — WCAG 3 is not ready yet](https://yatil.net/blog/wcag-3-is-not-ready-yet)
- [APCA in a Nutshell](https://git.apcacontrast.com/documentation/APCA_in_a_Nutshell.html)
- [axe-core issue #3325 — APCA support](https://github.com/dequelabs/axe-core/issues/3325)
- [arXiv 2501.03572 — From Code to Compliance (ChatGPT case study)](https://arxiv.org/html/2501.03572v2)
