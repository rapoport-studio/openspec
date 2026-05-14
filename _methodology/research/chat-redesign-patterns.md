# Chat redesign — modern UI patterns (source stream 1 of 4)

> 2026-05-14 · feeds [synthesis](chat-redesign-synthesis.md) · sister streams: [a11y](chat-redesign-a11y.md), [voice](chat-redesign-voice.md), [adjacent](chat-redesign-adjacent.md).
>
> Coverage: state-of-the-art chat UI patterns for AI assistants + team-collaboration chat, late 2025 / early 2026.

---

## 1. Layout patterns

- **Single-column-with-collapsible-left-rail is dominant.** ChatGPT, Claude.ai, T3 Chat, Perplexity, OpenWebUI, LibreChat all converge: left rail = conversation list + (optional) projects/folders; center = messages; composer pinned to bottom.
- **Multi-pane is migrating into product surfaces, not chat.** Claude.ai Artifacts opens a right-side live pane beside chat for code/docs/diagrams; Cursor's Composer pairs chat with file diffs; Claude Code Desktop (Apr 2026) added drag-and-drop multi-pane with sidebar + chat + terminal + preview.
- **Header is intentionally near-empty.** T3 Chat's redesign got buzz partly for this — model picker top-left, share/settings top-right, that's it. ChatGPT 2026 collapsed its model picker to three choices (Instant / Thinking / Pro) after years of complaints about confusion.
- **Linear Asks / agent threads break form deliberately:** AI replies live *inline in issue comments*, not in a dedicated chat surface. Agents are workspace members; @-mention triggers them.
- **Notion AI** sits as a circular ambient button (bottom-right), expanding to a context-aware sheet rather than a full chat surface — explicitly designed to "not break stride."
- **Perplexity** is the outlier — answer-first, source-cards inline, predicted follow-ups always-on at the bottom; minimal sidebar.

## 2. Per-message actions

- **Consensus set:** copy, regenerate, edit (user msg), thumbs up/down. Branch and share-link are emerging but not standard.
- **Hover-revealed is now standard on desktop**; ChatGPT's May 2026 restructure moved retry into a hover-only floating bar and users actively complained ("missing it entirely because it only renders on mouse-over") — cautionary tale, don't fully hide the most-used action.
- **Mobile: tap-to-reveal under the message bubble** wins over long-press in modern apps; long-press is reserved for native message apps. T3 Chat keeps actions visible on mobile.
- **Edit-and-resend is the killer feature** for assistant-grade chats; Claude.ai and ChatGPT both retain a tree of branches when you edit. T3 Chat and OpenWebUI support branching; LibreChat exposes it more explicitly.
- **Successful break from convention:** Perplexity surfaces sources/citations as *primary* per-message actions (not hidden) — the entire UI rewards verifying.

## 3. Composer patterns

- **Auto-grow textarea, max ~40% viewport then scroll** — standard everywhere (Vercel AI Elements `PromptInput`, assistant-ui, prompt-area). Enter = send, Shift+Enter = newline; on mobile Enter = newline, send button required.
- **Slash commands** leave a *directive chip* in the composer for audit (assistant-ui's pattern). `@` for mentions/agents (Linear-style), `/` for actions, `#` for tags is becoming the trio.
- **Attachments via drag-drop on the composer + paste-image-from-clipboard** is table stakes since 2024.
- **Model picker placement: top of composer or top-left header.** Burying in settings is widely criticized. T3 Chat does an inline dropdown right above the textarea — praised for speed of switching mid-thread.
- **Send button morphs into Stop button during streaming.** Modern consensus — same button, different icon/color. Vercel AI SDK ships this pattern by default.
- **Footer chrome (token count, model name, cost):** *muted, single line below composer*. Hide by default in consumer-leaning apps; show in dev/power-user apps. For B2B workspace: default muted but visible.

## 4. Streaming response UX

- **Word-level streaming with a soft blinking caret** at the tail is the dominant feel. Pure token-level looks janky (visible mid-word breaks) — most apps now buffer to word boundaries.
- **Block-level deferred render for markdown/code.** Production apps buffer until a markdown block (code fence, list, table) closes, then commit it — prevents flicker. FlowToken, llm-ui, Vercel AI Elements all do this.
- **Subtle fade/blur-in animation per word** is the current "feels premium" choice (Claude.ai, T3 Chat). Typewriter-per-character is now considered dated.
- **Abort must be prominent during streaming, never in a menu** — consensus.
- **Regenerate-with-different-model** is now an expected affordance on the regenerate dropdown.

## 5. Empty / loading / error states

- **First-message empty state: 3–6 suggestion chips OR blank canvas with placeholder, not both.** ChatGPT removed its big example cards in 2024 and moved to a quieter placeholder; Claude.ai uses 4 chips. Perplexity uses a Discover feed instead. T3 Chat: nearly blank.
- **Anti-pattern called out repeatedly:** "infinite suggestion chips" and generic placeholders that paralyze users.
- **Loading: skeleton lines under an assistant avatar/sparkle**, never a spinner inside the bubble.
- **Errors must differentiate** safety blocks vs rate-limit vs network vs context-limit — generic "Something went wrong" is widely criticized. Always preserve unsent prompt.
- **Rate-limit messaging:** Anthropic and Claude.ai surface a countdown + upgrade CTA inline in chat — now standard.

## 6. Community sentiment (2025–2026)

The loudest meta-thread: **"chat is the wrong UI."** Allen Pike's *Post-Chat UI* (2025) is widely cited — quotes Amelia Wattenberger's "We made painting feel like typing, but we should have made typing feel like painting." Argues chat should be a *fallback* mode, not the primary surface. Echoed on HN's "Chat is a bad UI pattern for development tools."

Maggie Appleton, Linus Lee, Geoffrey Litt spoke at the AI×UX meetup at Notion HQ — recurring themes: writing daemons (Appleton), instrumental vs engaged interfaces (Lee), direct manipulation > chatbox (Litt). Roger Wong's Litt malleable software piece (Nov 2025) and the Latent Space Linus Lee episode are the citable artefacts.

Simon Willison mostly writes about agentic coding patterns in 2025–2026, not chat UI per se; he endorses Wattenberger's "boo chatbots" stance.

Wattenberger's *Intent* project (post-IDE) is the most-shared "what comes after chat" demo — workspace per task, agents + markdown notes + terminals + spec, no central chat.

## 7. Anti-patterns (concretely criticized 2025–2026)

1. Buried/confusing model pickers (ChatGPT's pre-2026 6-model dropdown).
2. Hover-only critical actions (ChatGPT May 2026 retry-button hiding).
3. Empty-prompt-field-with-generic-placeholder.
4. No abort button / stop in a submenu.
5. Over-disclaimering — medical disclaimers dropped from 26.3% (2022) to 0.97% (2025).
6. Generic error messages.
7. Forcing every action through chat.
8. Adaptive/generative UI that changes between sessions.

## 8. Minimal-design exemplars

- **T3 Chat** — most-cited 2025 minimalism win. Theo Browne's redesign (March 2025) emphasized: fast model switch in header dropdown, near-empty chrome, keyboard-first navigation, hotkeys for everything, local-first architecture. Liked specifically for what it removed.
- **Claude.ai** — got design points for restraint (until Artifacts added complexity).
- **Perplexity** — minimal in a different axis: answer-first, sources inline, predictive follow-ups.
- **Linear (Asks/agents)** — minimal because chat is *absent as a destination*.
- **Notion AI** — collapse to a button until summoned.

## Sources

- [Post-Chat UI — Allen Pike (2025)](https://allenpike.com/2025/post-chat-llm-ui/)
- [Why Chatbots Are Not the Future — Amelia Wattenberger](https://wattenberger.com/thoughts/boo-chatbots/)
- [Design Patterns for AI Interfaces — Smashing Magazine, Jul 2025](https://www.smashingmagazine.com/2025/07/design-patterns-ai-interfaces/)
- [AI Chat UI Best Practices — thefrontkit](https://thefrontkit.com/blogs/ai-chat-ui-best-practices)
- [AI Chat Interface Pattern — uxpatterns.dev](https://uxpatterns.dev/patterns/ai-intelligence/ai-chat)
- [Chat is a bad UI pattern for dev tools — HN](https://news.ycombinator.com/item?id=42934190)
- [T3 Chat redesign launch — Theo Browne](https://www.linkedin.com/posts/t3gg_t3-chat-is-now-the-cheapest-fastest-and-activity-7307892576580161536-eYBv)
- [Claude Code Desktop redesign — tbreak](https://tbreak.com/claude-code-desktop-redesign-2025/)
- [Linear for Agents](https://linear.app/agents)
- [LibreChat vs Open WebUI 2025 — Elestio](https://blog.elest.io/the-best-open-source-chatgpt-interfaces-lobechat-vs-open-webui-vs-librechat/)
- [Notion AI Assistant case study — BUCK](https://buck.co/work/notion-ai)
- [Perplexity UI study — SaaSUI](https://www.saasui.design/application/perplexity-ai)
- [Vercel AI SDK chatbot docs](https://ai-sdk.dev/docs/ai-sdk-ui/chatbot)
- [Streaming UI guide — thefrontkit](https://thefrontkit.com/blogs/what-is-streaming-ui-in-ai-applications)
- [ChatGPT retry-button removal complaint](https://www.aiqnahub.com/chatgpt-web-ui-retry-button-removed/)
- [Medical disclaimers declining — MIT Tech Review](https://www.technologyreview.com/2025/07/21/1120522/ai-companies-have-stopped-warning-you-that-their-chatbots-arent-doctors/)
